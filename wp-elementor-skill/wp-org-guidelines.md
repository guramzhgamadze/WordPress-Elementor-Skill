# WordPress.org Plugin Directory — Guidelines, Plugin Check & Review Reality

> **When to read this file:** Any plugin destined for the **wordpress.org directory**. These are
> the *policy* rules that get a plugin accepted or rejected — distinct from `field-notes.md`
> (the practical "how it breaks" gotchas) and `php-standards.md` (the security *implementation*).
> Read all three for a submission. Canonical, always-current source — verify against it before
> relying on any summary:
> developer.wordpress.org/plugins/wordpress-org/detailed-plugin-guidelines/

---

## The 18 Detailed Plugin Guidelines (paraphrased — actionable form)

Reviewers cite these by number ("this fails Guideline 8"). Keep the numbering.

1. **GPL-compatible license.** All code, bundled libraries, data, and images must be
   **"GPLv2 or later" or another GPL-compatible license.** No "all rights reserved" assets.
2. **The developer is responsible for all files** in the plugin and for following the
   guidelines. Deliberately circumventing a rule is itself a violation.
3. **A stable version must be available from the directory** (your SVN `Stable tag`). Don't ship
   the real code only from your own site while the directory hosts a stub.
4. **Human-readable source — no obfuscation.** No packed/minified-only/encoded PHP whose source
   isn't included or clearly linked. JS/CSS may be minified **if** the unminified source is
   included or referenced. p,a,c,k,e,r and similar are rejected.
5. **No "trialware."** Functionality may not be crippled, time-locked, or disabled pending
   payment, and the plugin may not be a sandbox-only/demo. (Up-sells to a Pro version are fine;
   nagging or disabling core features is not.)
6. **SaaS is allowed** when the third-party service provides *substantial* functionality. Pure
   **license-key validation servers** and **storefront-only** plugins are not — the service must
   do real work, not just gate the plugin.
7. **No phoning home / tracking without explicit, opt-in consent.** Don't contact external
   servers or load remote assets on activation by default. Telemetry must be **opt-in**, clearly
   disclosed, with a privacy policy. (A genuine SaaS the user signed up for is the exception.)
8. **No executable code pulled from remote servers.** No self-updating from outside wordpress.org,
   no fetch-and-`eval()`, no loading PHP from a CDN. The directory is the only update source.
9. **No illegal or dishonest actions** — black-hat SEO, fake/incentivized reviews, sockpuppets,
   stolen code, botnets, crypto-mining, spam.
10. **No required credits / "Powered by" links.** Any front-end credit/attribution link must be
    **opt-in and default to OFF.**
11. **Don't hijack the admin.** Notices must be relevant, contextual, and **dismissible**; no
    persistent dashboard takeovers or unrelated upsell banners. Admin notices should auto-clear
    once the condition is resolved.
12. **No readme spam.** ≤ **5 tags**, no competitor/brand keyword stuffing, no affiliate-link
    farms, no black-hat SEO in the readme or other public-facing text.
13. **Use WordPress's bundled libraries** — `jquery`, `SimplePie`, `PHPMailer`, etc. — rather than
    bundling your own copy. (Ties to the skill's "no jQuery unless it's a WP-core dependency" rule
    and "declare `'swiper'` as a dependency, don't ship your own.")
14. **Don't abuse the SVN repo with frequent junk commits.** SVN is a *release* repo; commit
    deploy-ready code, not dozens of "wip/typo" revisions.
15. **Increment the version on every release** so users get the update notification. (See the
    lockstep version-bump checklist in `field-notes.md` §10.)
16. **The plugin must be complete and functional at submission.** Names/slugs can't be reserved
    for future work; an empty or stub plugin is rejected.
17. **Respect trademarks.** A slug may not **start** with a trademarked term you don't own. Use
    `"… for Brand"` / `"Tool for Brand"`, never `"Brand Tool"`. (Affects `get_name()` /
    text-domain / slug choices.)
18. **The directory reserves the right** to update guidelines, disable a plugin for user safety,
    grant case-by-case exceptions, and push emergency security fixes without author consent.

---

## Plugin Check (PCP) 2.0.0 — the reviewer's own tool

The official plugin (by the WordPress.org team) that runs **most of the checks used for new
submissions**. Requires WP 6.3+ / PHP 7.4+. **Run it until your own code is 0 findings before
every submission and resubmission** (standing project rule). Passing is necessary but **not
sufficient — human review is still mandatory.**

**Check categories:**
- **Plugin Repository Requirements** — directory-guideline compliance (headers, readme, naming,
  no disallowed functions, trademark/branding).
- **Security** — sanitization, escaping, nonces, SQL prep, file operations.
- **Performance** — enqueue patterns, asset optimization.
- **Accessibility** — accessible-markup checks.
- **Code Standards** — PHPCS (WordPress Coding Standards) — emits the `WordPress.Security.*`,
  `WordPress.NamingConventions.*`, etc. codes.
- **Internationalization** — correct use of i18n functions and a single, matching text domain.

**How to run:**
```bash
# WP-CLI (preferred for CI / scripted checks):
wp plugin check <your-plugin-slug>
wp plugin check <slug> --format=json --exclude-directories=includes/lib
```
- **Static checks run by default.** **Runtime checks** require loading the plugin's `cli.php`
  first: add `--require .../cli.php` before WordPress boots.
- Admin UI: **Tools → Plugin Check**.
- Since **October 2025, Plugin Check also runs automatically on every plugin update**, not just
  new submissions — a security regression in an update can get the plugin flagged/closed.

See `field-notes.md` §3 for how PCP flags `echo $var` / `echo $this->method()`, the
`phcs:ignore` philosophy, and the small set of *justified* false positives (core hook names,
cache-opt-out constants, excluded vendored libs).

---

## The review process — what actually happens (2025–2026)

- **Reviews are AI-assisted but human-decided.** Automated first-pass tooling handles the bulk
  (the team added 80+ internal checks and Plugin Check auto-scans), but a person makes the call —
  "these tools support reviewers; they don't replace them."
- **Queue is typically days to ~a week** (submission volume surged to hundreds/week through 2026;
  the team scaled reviewers to keep turnaround short). Don't assume instant approval; build the
  submission cleanly the first time.
- **Author responsiveness strongly correlates with approval.** Approved plugins average **several
  review cycles**, and a large share of rejected ones simply **never replied to the reviewer's
  email**. When a reviewer writes, **reply promptly and address every point** — silence is the
  most common path to rejection.
- **Fix the source, don't argue or suppress.** Reviewers grep for hand-wavy `phcs:ignore`
  justifications and known evasion patterns (see `field-notes.md` §2–3).
- **The review is multi-round and ESCALATES.** Round 1 is commonly an automated
  **AUTOPREREVIEW** (bot + AI-generated notes); later rounds are a **human reviewer** who catches
  what the bot missed and looks harder each pass. Fix the whole *class* of a flagged issue and
  self-audit for other occurrences — reviewers state "we may not share all cases."
- **Reply in the SAME email thread** — concise, no change-logs, no AI filler. Reviewers re-review
  the **whole plugin**, they don't diff your changes; one complete, well-tested update per round
  beats many partial ones (each round can take days to weeks).

---

## Hard lines from real 2026 reviews — not spelled out in the guidelines

- **No user-authored SQL over any endpoint — guards cannot fix it.** A tool running free-form
  caller-supplied `SELECT` was rejected even with SELECT-only checks, credential-column blocks,
  secret redaction and admin auth: free-form SQL **cannot be `prepare()`d** (the entire string is
  the input — no placeholders to bind), and exposing arbitrary DB reads over a remote,
  token-authenticated REST endpoint is an aggravating factor. The redesign that passed: a
  **structured query the plugin assembles itself** — **identifiers** (table, columns) validated
  against the **live schema** (`SHOW TABLES` / `DESCRIBE`) plus `^[a-zA-Z0-9_]+$`, then
  interpolated in backticks; **values** bound via `$wpdb->prepare()`. Rule of thumb for ANY
  user-input-to-DB feature: **whitelist identifiers, parameterize values.** If a feature only
  works by running user-authored SQL, it does not belong in a wp.org plugin (ship it GitHub-only,
  filter-gated, instead).
- **Never use `AUTH_KEY` / auth salts outside core authentication** — not even to derive an
  opaque one-way HMAC identifier. Reviewers flag it; generate a dedicated plugin secret option
  instead.
- **Document every external service in a `== External services ==` readme section** — what it is,
  what data is sent, when, plus ToS/privacy links. Required **even when the service connects
  INBOUND** and the plugin never phones out.
- **Automated nonce flags on protocol endpoints are answered, not "fixed".** OAuth
  `/authorize` + `/token` (secured by PKCE + single-use codes) legitimately have no WP nonce —
  explain that in the reply; do **not** bolt on a meaningless nonce. (Admin-facing forms still
  verify `check_admin_referer()` + capability, always.)
- **A standalone HTML page (echoed + `exit`, no `wp_head()`) still may not emit a raw `<style>`
  or `<link>`.** The pattern that satisfies both Plugin Check and the reviewer:
  `wp_register_style()` + `wp_enqueue_style()`, then print by handle with
  `wp_print_styles( 'my-handle' )` inside your own `<head>` — and move inline `style=""`
  attributes into that stylesheet too.

---

## Required headers & readme (cross-reference)

A compliant submission needs, in lockstep (full checklist in `field-notes.md` §10):
- **Main file header:** `Plugin Name`, `Version`, `Requires at least`, `Requires PHP`,
  `License: GPL-2.0-or-later`, `Text Domain`, and `Requires Plugins:` if it depends on Elementor.
- **`readme.txt`:** `Stable tag` (must match the released version), `Requires at least` (the
  REAL feature floor — see the compatibility-gate note in `debugging.md` §1), `Tested up to`
  (**must equal the current WP major** or the readme check errors), `Requires PHP`, `License`,
  ≤ 5 tags, a `== Changelog ==`, and an `== Upgrade Notice ==` (< 300 chars each).
- **Listing assets** (`screenshot-N.png` + captions, `banner-*`, `icon-*`) live in SVN
  `/assets/`, **not** in the plugin zip.

> 🚀 **Actually deploying to the directory** (the `svn co` → copy to `trunk/` → `svn cp trunk
> tags/X.Y.Z` → `svn ci` release dance, plus updating `/assets/`) is a Subversion task — see the
> **`svn/`** sub-bundle (`svn/svn.md`), which has a WordPress.org-specific worked example.

> Source of record (re-check before each submission, guidelines do change):
> developer.wordpress.org/plugins/wordpress-org/detailed-plugin-guidelines/ ·
> wordpress.org/plugins/plugin-check/ · make.wordpress.org/plugins/ (review-team announcements)
