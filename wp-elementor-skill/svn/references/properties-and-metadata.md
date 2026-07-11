# Properties and Metadata

Deep reference for SVN's property system: versioned metadata attached to
files, directories, and revisions, separate from file content itself.

## Table of contents
- Property basics
- Manipulating properties (the command set)
- Revision properties (unversioned)
- Properties and the normal workflow (status, diff, conflicts)
- Automatic property setting
- svn:ignore in depth
- svn:externals in depth
- Locking (svn lock/unlock, breaking/stealing, svn:needs-lock)
- Changelists

## Property basics

Properties are two-column name/value tables attached to a versioned file or
directory (or, separately, to a whole revision). Names must be ASCII;
values can be anything, including multi-line or binary data. Names starting
with `svn:` are reserved for Subversion's own use — don't invent custom
properties in that namespace.

Properties are versioned exactly like file content: they're part of your
local modifications until committed, they show up in `svn status`/`svn
diff`, they merge and can conflict, and `svn revert` undoes property
changes along with content changes.

Custom properties are easy to *set* but hard to *search* — there's no
indexed lookup. Finding a custom revision property means a linear walk
across all revisions (`svn log --with-all-revprops --xml`); finding a
custom versioned property means a recursive `svn propget` across the whole
working copy. For metadata you'll need to search later, many teams instead
put a structured tag in the commit log message (e.g. `Issue(s): IZ2376`)
rather than a property.

## Manipulating properties

```bash
svn propset copyright '(c) 2026 Example' file.php     # set from a literal value
svn propset license -F LICENSE.txt file.php            # set from a file (multi-line/binary safe)
svn propedit copyright file.php                        # open $EDITOR to view+edit current value
svn proplist file.php                                  # list property NAMES on a path
svn proplist -v file.php                                # list names AND values
svn propget copyright file.php                          # print one property's value
svn propdel copyright file.php                          # remove a property entirely
```
Prefer `propedit` over `propset` when practical — it shows you the current
value before you overwrite it, which matters most for revision properties
(see below) where a mistake can't be recovered. Property commands accept
multiple targets/globs at once: `svn propset copyright '...' src/*`.

Note: setting a property to an empty string with `propset` does **not**
delete it — `svn propdel` is the only way to actually remove one.

Property names must start with a letter, `:`, or `_`; after that, digits,
hyphens, and periods are also allowed.

## Revision properties (unversioned)

Every revision automatically gets `svn:author`, `svn:date`, `svn:log`, etc.
Unlike file/directory properties, **revision properties are not
versioned** — editing or deleting one destroys the previous value with no
undo. The most common legitimate use is fixing a typo in a commit message
after the fact:
```bash
svn propset svn:log "Corrected commit message." -r11 --revprop
```
By default, servers disable revprop editing entirely (it requires the
administrator to enable it via the `pre-revprop-change` hook) — see
`repository-admin.md`.

## Properties and the normal workflow

`svn status` shows `M` in the *second* column (not the first) when only
properties changed, not content:
```
 M      calc/button.c
```
`svn diff` shows property changes in a distinct, non-patch-compatible
format:
```
Property changes on: calc/button.c
___________________________________________________________________
Added: copyright
## -0,0 +1 ##
+(c) 2026 Example
```
The standalone `patch` program ignores this section entirely (it only
understands content diffs); SVN's own `svn patch` subcommand (1.7+) does
apply property changes from a diff. One property, `svn:mergeinfo`, gets
special human-readable formatting in diffs specifically so merge output is
legible — treat it as managed exclusively by `svn merge`, not something to
hand-edit.

**Property conflicts** happen just like content conflicts, reported the
same way during `svn update`, and leave a `.prej` file explaining the
clash. Resolve with `svn resolve --accept working` after fixing the
property value by hand.

## Automatic property setting

On `svn add` / `svn import`, SVN tries to help:
- Sets `svn:executable` automatically if the OS execute bit is set on the
  file (non-Windows filesystems).
- Guesses `svn:mime-type` — using a configured MIME-type-mapping file if
  you've set one up, otherwise heuristics (including `libmagic` where
  available), falling back to `application/octet-stream` if it looks
  non-textual.

You can also configure pattern-based auto-props in the runtime config
(`config` file, `[auto-props]` section) so that, say, every added `*.jpg`
automatically gets `svn:mime-type = image/jpeg`, or every `*.cpp` gets
`svn:eol-style = native` and `svn:keywords = Id`. That `config` file lives
at `~/.subversion/config` on Unix-like systems or
`%APPDATA%\Subversion\config` on Windows (created the first time the `svn`
client runs). **The auto-props section is inert by default** — you also
have to set `enable-auto-props = yes` in the same file's `[miscellany]`
section, or none of the patterns you configure will actually fire; this is
the classic reason auto-props "isn't working." This has to be configured
per-client — there's no way for a server to force these defaults onto
everyone connecting to it (though a `pre-commit` hook can *reject* commits
that don't set expected properties).

## svn:ignore in depth

Two separate mechanisms, both pattern-based (shell-style globs: `?`, `*`,
`[...]`):

1. **`global-ignores`** runtime config option — whitespace-delimited
   patterns, applies on your machine to every working copy (e.g. editor
   backup files like `*~`).
2. **`svn:ignore` property** — set per-directory, versioned, so it travels
   with the repository and applies to everyone who checks it out. It does
   **not** cascade to subdirectories — set it separately wherever needed.
   These patterns are *appended* to the global list, not a replacement.

```bash
svn propedit svn:ignore calc/     # opens editor; enter one pattern per line
svn propget svn:ignore calc/      # view current patterns
```
Migrating from CVS: `svn propset svn:ignore -F .cvsignore .` imports a
`.cvsignore` file directly (note: unlike CVS, SVN doesn't support `!` to
reset the ignore list).

Ignore patterns only affect the one-time decision of what `svn add` / `svn
import` sweep in, and what `svn status` reports by default (`--no-ignore`
shows ignored items with an `I` marker). Once a file is actually under
version control, ignore patterns have **no further effect** — SVN will
always track and commit changes to it regardless of a matching pattern.

Gotcha: shell wildcards get expanded by your shell *before* SVN sees them,
so `svn add *` bypasses `svn:ignore` for anything the shell's glob
matches. Use `svn add --force --depth files .` for a controlled bulk-add
that still honors ignore patterns.

## svn:externals in depth

`svn:externals` maps a local subdirectory to a URL (optionally pinned to a
revision) of another versioned directory — anyone who checks out the
parent automatically gets the external pulled in too. Common use: vendored
third-party code or shared assets.

Modern (1.5+) syntax, one mapping per line, checkout-argument order:
```
-r148 http://svn.example.com/skinproj third-party/skins
      http://svn.example.com/sounds   third-party/sounds
```
Relative URL forms (1.5+) avoid hardcoding the server:

| Prefix | Relative to |
|---|---|
| `../` | the directory the property is set on |
| `^/` | the repository root |
| `//` | the URL scheme of the directory the property is set on |
| `/` | the server root |

Set/edit with `propedit` (multi-line, so avoid `propset`):
```bash
svn propedit svn:externals calc/
```

**Sharp edges worth knowing before you rely on externals:**
- Pin to an explicit revision unless you deliberately want externals to
  float to `HEAD` — otherwise everyone's checkout silently picks up
  upstream changes you don't control, and backdating your working copy
  won't correctly backdate an unpinned external.
- External working copies are genuinely separate, disjoint working
  copies. Committing in the parent working copy does **not** recurse into
  externals — you must `svn commit` inside the external directory itself.
- Absolute URLs in old-style definitions break if you move/rename the
  directory the property lives on, or if you `svn relocate` the parent —
  externals don't automatically follow either.
- File externals (single-file, not directory) can't be moved or deleted
  directly (edit the `svn:externals` property instead), and can only
  reference a file in the *same* repository.
- `--ignore-externals` disables external processing for `checkout`,
  `update`, `switch`, `export`, and `status` when you need to skip them.

## Locking

Locking exists for the one case copy-modify-merge handles badly:
essentially-unmergeable files (binary images, fonts, compiled assets).
SVN's lock is one of **three unrelated things called "lock" in SVN** —
don't confuse them:
1. **Repository locks** (this section) — mutual exclusion between users.
2. **Working copy locks** — internal, prevents two SVN client processes
   from stepping on the same working copy; shown as `L` in `svn status`;
   cleared with `svn cleanup` (see `troubleshooting.md`).
3. **Database locks** — internal to the Berkeley DB backend; can "wedge" a
   repository if a process dies mid-transaction (see `repository-admin.md`).

```bash
svn lock file.jpg -m "Editing for tomorrow's release."   # claim exclusive right to commit
svn status                # shows K (locKed) next to the file
svn info file.jpg          # shows lock token, owner, comment, timestamp
svn unlock file.jpg        # release voluntarily
```
While locked by someone else, your commits touching that file are rejected
until they unlock or you break the lock. `svn status -u` shows an `O`
(locked by Other) next to affected files.

**Breaking vs. stealing:** by default anyone can release anyone else's
lock (not just the owner or an admin) — this is a deliberate "locks are a
communication tool, not a security boundary" design choice, though
`pre-lock`/`pre-unlock` hooks can enforce stricter policy if a team wants
it.
```bash
svn unlock --force https://example.com/repo/file.jpg    # break someone else's lock
svn lock --force file.jpg                                # steal: break + relock atomically
```
An admin can also inspect/remove locks directly on the server without a
working copy: `svnadmin lslocks REPO_PATH`, `svnadmin rmlocks REPO_PATH
PATH`.

By default, **committing releases every lock your commit touches — even
ones on files you didn't actually change** if they were part of the
targets you passed to `svn commit`. Use `--no-unlock` on the commit to keep
holding locks you still need.

**`svn:needs-lock`** is a property, not a lock itself — attach it to a
file (any value; only presence matters) and SVN makes the file read-only
on checkout/update until someone locks it, then read/write while locked.
It's a *reminder* mechanism (many editors refuse to save a read-only file,
prompting the user to go lock it first) — it doesn't stop anyone who edits
around the read-only bit, and it doesn't make the repository require a
lock to accept a commit.

## Changelists

A lightweight, purely local grouping mechanism — not synced to the
repository — for organizing pending edits into named buckets so you can
`commit`/`diff`/`revert` a subset of your working copy at once:
```bash
svn changelist my-bugfix file1.php file2.php
svn commit --changelist my-bugfix -m "Fix the thing"
```
Useful when you have several unrelated edits mixed together in one working
copy and want to commit (or review) them separately without juggling
multiple checkouts. Two limits worth knowing before relying on them:
changelists only apply to files, not directories, and a file can only
belong to one changelist at a time (assigning it to a new one silently
moves it out of the old one). Also, **committing a file normally clears
its changelist assignment** once the commit succeeds — pass
`--keep-changelists` to the commit if you want the label to stick around
for next time.
