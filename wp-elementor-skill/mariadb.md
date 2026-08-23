# MariaDB / MySQL — the database layer for WordPress plugins

> **When to read this file:** Creating or altering a **custom table**, writing **direct `$wpdb`
> queries**, diagnosing a **slow query**, choosing **column types or indexes**, or hitting a
> charset / collation / "Specified key was too long" error. Distilled from the official **MariaDB
> Knowledge Base** (mariadb.com/kb) and the WordPress plugin handbook.
>
> **Scope:** WordPress runs on **both MariaDB and MySQL**, and a distributed plugin does not get to
> choose which. Everything here is written for that reality — §6 lists the differences that
> actually break portable SQL. For the `$wpdb->prepare()` security rules and the Plugin Check
> sniffs that fire on direct queries, see **`php-standards.md`** and **`debugging.md`** §1; this
> file is about *schema and performance*, not escaping.

---

## 1. Versions — what WordPress actually needs

| | Recommended (wordpress.org/about/requirements) | Will still run |
|---|---|---|
| **MariaDB** | **10.11+** | 5.5.5+ (End of Life — a security liability) |
| **MySQL** | **8.0+** | 5.5.5+ (End of Life) |

- The recommended MariaDB floor was **raised from 10.6 to 10.11** — verify against the live page
  before quoting it; it moves.
- **Never assume the server version.** Shared hosts run old builds. Check at runtime before using
  anything version-dependent:
  ```php
  global $wpdb;
  $version   = $wpdb->db_version();              // e.g. "10.11.6" — normalised by WP
  $is_maria  = false !== stripos( $wpdb->db_server_info(), 'mariadb' );
  ```
- Declare nothing about the DB in your plugin header — there is no `Requires MySQL`. If you need a
  modern feature, **detect it and degrade**, don't document a requirement nobody reads.

---

## 2. Character sets & collations — and the 191-character rule

**`utf8` is not UTF-8.** MariaDB has two implementations:

| Charset | Bytes/char | Stores |
|---|---|---|
| `utf8mb3` (historically aliased as `utf8`) | 1–3 | Latin, European, Middle-Eastern scripts — **no supplementary characters** |
| **`utf8mb4`** | 1–4 | **All of Unicode** — emoji, rare CJK, mathematical symbols |

In MariaDB 10.6+, `utf8` remains an alias for **`utf8mb3`** by default (changeable via the
`old_mode` system variable). So a column declared `utf8` **silently cannot store an emoji** — the
classic "user pasted 🎉 and the row truncated or errored" bug.

> ✅ **Never write a charset by hand. Use WordPress's:**
> ```php
> $charset_collate = $wpdb->get_charset_collate();   // e.g. "DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_520_ci"
> $sql = "CREATE TABLE {$table_name} ( … ) {$charset_collate};";
> ```
> It reads `DB_CHARSET` / `DB_COLLATE` from `wp-config.php` and matches the rest of the install —
> which also keeps your table **JOIN-able** with core tables. Joining columns of *different*
> collations triggers `Illegal mix of collations` at runtime, not at `CREATE TABLE` time.

### ⚠️ The 191-character rule — why WordPress writes `varchar(191)`

InnoDB caps an **index key** by page size (official limits):

| InnoDB page size | Max index key length |
|---|---|
| 4 KB | 768 bytes |
| 8 KB | 1,536 bytes |
| **16 KB (the default)** | **3,072 bytes** |

Older row formats (`COMPACT` / `REDUNDANT`) cap a single column prefix at **767 bytes**. At 4
bytes per character under `utf8mb4`:

```
767 ÷ 4 = 191.75  →  191 characters
```

That is the entire reason WordPress core declares indexed string columns as **`varchar(191)`**
rather than 255. Follow it:

```sql
-- ❌ errors on older row formats: "Specified key was too long; max key length is 767 bytes"
email varchar(255) NOT NULL,
KEY email (email)

-- ✅ safe everywhere
email varchar(191) NOT NULL,
KEY email (email)

-- ✅ or index only a prefix when you need the full column length
url varchar(2048) NOT NULL,
KEY url (url(191))
```

Modern `DYNAMIC` row format on 16 KB pages allows 3,072 bytes, so 255 chars *usually* works — but
"usually" is not a basis for a distributed plugin. **Index at 191, or index a prefix.**

---

## 3. Custom tables — `dbDelta()` and its unforgiving syntax

Only create a custom table when post types + meta genuinely don't fit (high-volume logs, analytics,
queues). You lose the REST API, revisions, caching and every core query helper.

```php
function myplugin_install_table(): void {
    global $wpdb;

    $table_name      = $wpdb->prefix . 'myplugin_events';   // ALWAYS use the prefix
    $charset_collate = $wpdb->get_charset_collate();

    $sql = "CREATE TABLE {$table_name} (
        id bigint(20) unsigned NOT NULL AUTO_INCREMENT,
        user_id bigint(20) unsigned NOT NULL DEFAULT 0,
        event_type varchar(191) NOT NULL DEFAULT '',
        payload longtext NULL,
        created_at datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
        PRIMARY KEY  (id),
        KEY user_id (user_id),
        KEY event_type_created (event_type, created_at)
    ) {$charset_collate};";

    require_once ABSPATH . 'wp-admin/includes/upgrade.php';
    dbDelta( $sql );

    update_option( 'myplugin_db_version', MYPLUGIN_DB_VERSION );
}
register_activation_hook( __FILE__, 'myplugin_install_table' );
```

> 🔴 **`dbDelta()` parses your SQL with regexes — its formatting rules are absolute.** Break one and
> it fails *silently* or re-runs `ALTER TABLE` on every load:
> - **One field per line.** No exceptions.
> - **Two spaces** after `PRIMARY KEY` — `PRIMARY KEY  (id)`. This is the single most common bug.
> - Use **`KEY`**, never `INDEX`. Include **at least one** `KEY`.
> - `KEY` → one space → key name → space → `(column)`.
> - **No backticks or apostrophes** around field names.
> - Field **types lowercase**, SQL keywords **uppercase**.
> - Every type that accepts a length **must state it** — `bigint(20)`, not `bigint`.

**Schema upgrades:** `register_activation_hook()` does **not** fire on plugin update (since WP 3.1).
Version-gate the migration on a normal request instead:

```php
add_action( 'plugins_loaded', function (): void {
    if ( get_option( 'myplugin_db_version' ) !== MYPLUGIN_DB_VERSION ) {
        myplugin_install_table();   // dbDelta() diffs and ALTERs — safe to re-run
    }
} );
```

`dbDelta()` **adds** columns and indexes; it never drops them. Removing a column is a deliberate,
hand-written `ALTER TABLE` guarded by a version check.

> ⚠️ **`CREATE TABLE` is DDL — Plugin Check flags it** (`DirectDatabaseQuery.SchemaChange`, plus
> `.DirectQuery` and `.NoCaching`). Use the **block form** `phpcs:disable` / `phpcs:enable` with a
> truthful justification — see `debugging.md` §1, which lists every sniff a `$wpdb` call can fire.

---

## 4. Indexes — the 80% that matters

| Type | Purpose |
|---|---|
| **PRIMARY KEY** | Unique + `NOT NULL`, one per table. InnoDB **appends the primary key to every secondary index**, so keep it narrow — an `AUTO_INCREMENT` integer, never a long string. |
| **UNIQUE** | Enforces uniqueness. Allows **multiple NULLs** — `NULL` never equals `NULL` in SQL, so a UNIQUE index will not stop repeated NULL rows. |
| **KEY** (plain) | Speeds lookups, enforces nothing. |

**Composite indexes follow the leftmost-prefix rule.** MariaDB can use the *leftmost part(s)* of a
multi-column index, not an arbitrary middle slice. Given `KEY event_type_created (event_type, created_at)`:

```sql
WHERE event_type = 'signup'                        -- ✅ uses the index
WHERE event_type = 'signup' AND created_at > ?     -- ✅ uses both columns
WHERE created_at > ?                               -- ❌ cannot use it — created_at is not leftmost
```
Order composite columns by **equality filters first, ranges last**.

**When an index will NOT be used:**
- `LIKE '%term%'` — a **leading** wildcard defeats a BTREE index entirely (full scan). Leading-anchored
  `LIKE 'term%'` is fine.
- A function or type-cast wrapped around the column (`WHERE DATE(created_at) = …`) — index the value
  you actually filter on, or store a pre-computed column.
- Very small tables, where a scan is genuinely cheaper.
- Low selectivity — an index on a two-value status column rarely earns its keep.

**Limits (InnoDB, official):** 1,017 columns per table · **64 secondary indexes** · **32 columns per
composite index** · row-size limit 65,535 bytes (BLOB/TEXT count only 9–12 bytes; their content lives
outside the row).

**Every index is a write tax.** Each `INSERT`/`UPDATE`/`DELETE` maintains every index on the table.
Index what you filter, join and sort on — not every column.

---

## 5. Diagnosing a slow query — `EXPLAIN` and `ANALYZE`

```sql
EXPLAIN SELECT * FROM wp_myplugin_events WHERE event_type = 'signup' ORDER BY created_at DESC;
```

Read these four columns first:

| Column | What it tells you |
|---|---|
| **`type`** | How rows are reached. Best→worst: `const`, `eq_ref`, `ref`, `range`, `index`, **`ALL`**. |
| **`key`** | Which index was actually chosen. **`NULL` = none used.** |
| **`rows`** | Estimated rows examined per lookup. Compare against the rows you expect back. |
| **`Extra`** | The warnings live here. |

🚩 **Red flags:**
- **`type: ALL`** — full table scan, "bad if the table is large."
- **`Using filesort`** — an extra sort pass; the `ORDER BY` isn't served by an index.
- **`Using temporary`** — a temp table was built, typical of `GROUP BY` / `DISTINCT` / `ORDER BY`.
- **`key: NULL`** on a large table — add the index, or fix the query so an existing one applies.

**`ANALYZE`** (not `EXPLAIN`) *actually runs* the query and reports **`r_rows`** (rows really
examined) and **`r_filtered`** beside the estimates. When the estimate and reality diverge wildly,
the optimizer is working from stale statistics — that mismatch is the finding.
`EXPLAIN FORMAT=JSON` gives the full plan for programmatic use.

> **In WordPress, reach for Query Monitor first** (see `debugging.md` §2) — it shows every query with
> its calling component and flags slow/duplicate ones, so you know *which* query to `EXPLAIN` before
> you open a SQL client. Enable `SAVEQUERIES` to capture them.

---

## 6. MariaDB ≠ MySQL — what breaks portable SQL

Your plugin will run on both. These are the officially-documented divergences that matter:

- **🔴 JSON is the big one.** MariaDB does **not** support MySQL's packed/native binary JSON type —
  it stores JSON as ordinary **TEXT/LONGTEXT** and *compares JSON as strings, not by JSON value*.
  MySQL compares by value. Consequences:
  - The `->` and `->>` shorthand operators are MySQL-native and arrived in MariaDB only much later.
    **Don't use them in distributed code** — prefer `JSON_EXTRACT()` / `JSON_UNQUOTE()`, present in
    both, and even then test on both servers.
  - **Never make JSON equality or sorting the server's job.** Two documents that MySQL calls equal
    may differ as strings in MariaDB.
  - ✅ **The portable WordPress answer:** store JSON (or a serialized array) in `longtext`, filter on
    real indexed columns beside it, and decode in PHP. If you need to query *inside* a document, you
    need a proper column — that is the schema telling you something.
- **`UNIX_TIMESTAMP()`** returns 6 decimal places on MariaDB and none on MySQL — a difference that
  silently breaks partitioning expressions and any code comparing the raw return value. Cast or round
  explicitly.
- **`EXTRACT(HOUR FROM …)`** follows the SQL standard on MariaDB (0–23); MySQL may return larger values.
- **Temporal storage formats** for `TIME` / `DATETIME` / `TIMESTAMP` differ at the byte level between
  MySQL 5.6 and MariaDB 10.0 — relevant to raw dumps and replication, not to normal SQL.
- **Not in MariaDB:** `CREATE TABLESPACE` for InnoDB, the X Protocol, packed JSON. Encryption and GTID
  implementations differ and are **not** cross-compatible.

> ✅ **Rule for distributed plugins: write plain, boring, standard SQL.** Stick to what both engines
> have had for years. Every clever engine-specific feature is a support ticket from the half of your
> users on the other one.

---

## 7. `sql_mode` — the setting that differs per server

MariaDB's default since 10.2.4: `STRICT_TRANS_TABLES, ERROR_FOR_DIVISION_BY_ZERO,
NO_AUTO_CREATE_USER, NO_ENGINE_SUBSTITUTION`.

- **`STRICT_TRANS_TABLES`** — invalid or missing data **aborts and rolls back** the statement rather
  than silently coercing it. An `INSERT` omitting a `NOT NULL` column with no default *fails* here
  and *succeeds* on a lenient server. Always give columns sane `DEFAULT`s (as in §3).
- **`ONLY_FULL_GROUP_BY`** — forbids selecting columns that are neither grouped nor aggregated.
  **MySQL enables it by default; MariaDB does not.** A `GROUP BY` query that works perfectly on your
  MariaDB dev box can error on a user's MySQL host. Select only grouped or aggregated columns.
- **`NO_ZERO_DATE`** — rejects `'0000-00-00'`. Legacy WordPress data contains these; never write one.

**The real lesson:** a permissive dev server hides bugs that a strict production server throws. Test
against strict mode, and never rely on silent coercion.

---

## 8. Full-text search — and why usually not

Full-text indexes work on **MyISAM, Aria, InnoDB and Mroonga**, on `CHAR` / `VARCHAR` / `TEXT`
columns only, and **not on partitioned tables**.

```sql
SELECT * FROM wp_myplugin_docs
 WHERE MATCH (title, body) AGAINST ('wordpress elementor' IN BOOLEAN MODE);
```

- **Natural language mode** (default) ranks by relevance; **boolean mode** supports `+` required,
  `-` excluded, `*` wildcard, `"…"` phrase — but does not rank by relevance.
- ⚠️ **Short words are silently ignored:** under 3 characters on InnoDB, under 4 on MyISAM
  (`innodb_ft_min_token_size` / `ft_min_word_length`). Users searching a 2-letter term get nothing
  back, with no error — a classic "search is broken" report.
- **Stopwords** (common words) are filtered out except in boolean mode.

> **For plugins, prefer `WP_Query`'s `s` parameter** — it respects post types, statuses and
> capabilities, and stays compatible with search plugins the user already runs. Reach for a
> `MATCH … AGAINST` index only on your **own** custom table, when `LIKE '%term%'` (which cannot use
> an index — §4) has become the measured bottleneck.

---

## 9. Querying from PHP — the short version

Full security rules live in **`php-standards.md`**; the schema-side essentials:

```php
global $wpdb;

// ✅ Values are BOUND. Identifiers (table/column names) are never bindable —
//    build them from $wpdb->prefix and your own whitelist, never from user input.
$rows = $wpdb->get_results(
    $wpdb->prepare(
        "SELECT id, event_type, created_at
           FROM {$wpdb->prefix}myplugin_events
          WHERE user_id = %d AND created_at > %s
          ORDER BY created_at DESC
          LIMIT %d",
        $user_id,
        $since,
        $limit
    )
);
```

- **`prepare()` needs at least one placeholder** — `prepare( $sql )` with none is the
  "missing argument 2" anti-pattern. An always-present `LIMIT %d` guarantees one.
- **`%s` / `%d` / `%f` are not quoted by you** — `prepare()` adds the quotes. `'%s'` is a bug.
- **`%i`** (WP 6.2+) safely interpolates an **identifier**; still validate it against a whitelist.
- **Never `SELECT *`** on a hot path — name the columns so a **covering index** can serve the query
  without touching the row.
- **Cache reads** (`wp_cache_get` / transients — see `php-standards.md`), and remember that under a
  persistent object cache transients are **not** in `wp_options`.
- Use `$wpdb->prefix` for per-site tables and **`$wpdb->base_prefix`** for a network-wide table on
  multisite.

---

> **Sources:** mariadb.com/kb — Unicode · InnoDB Limitations · Getting Started with Indexes ·
> EXPLAIN/ANALYZE · SQL Mode · Full-Text Index Overview · MariaDB vs MySQL Compatibility ·
> developer.wordpress.org/plugins/creating-tables-with-plugins/ · wordpress.org/about/requirements/
