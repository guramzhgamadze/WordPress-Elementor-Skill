# Repository Administration

Deep reference for creating, configuring, and serving SVN repositories.
Most of this is server-side/admin territory rather than day-to-day client
use — read `svn.md` first if you just need to *use* an existing
repository.

## Table of contents
- Creating a repository
- Repository layout for multiple projects
- Hooks
- The svnadmin / svnlook toolkit
- Choosing a server: svnserve vs. SSH vs. Apache
- Backup and disaster recovery basics

## Creating a repository

```bash
svnadmin create /var/svn/myrepo                  # default backend (FSFS)
svnadmin create --fs-type fsfs /var/svn/myrepo    # explicit FSFS
svnadmin create --fs-type bdb  /var/svn/myrepo    # Berkeley DB backend (legacy; FSFS is standard now)
```
`svnadmin` and `svnlook` are **server-side tools** — they take a local
filesystem path, not a URL, and can't operate across a network. (Passing a
URL, even a `file://` one, to `svnadmin` is a very common beginner
mistake.) The regular `svn` client is what takes URLs.

A fresh repository starts at revision 0 (an empty root directory) with a
single revision property, `svn:date`, set to creation time.

Don't hand-edit anything under a repository's data store directly —
`svnadmin` is the supported interface for everything short of examining
hook scripts and config files, which are meant to be edited directly.

## Repository layout for multiple projects

If one repository hosts several unrelated projects, give each its own
project root containing its own `trunk`/`branches`/`tags`, rather than one
shared set at the repository root:
```
svn list file:///var/svn/multi-project-repo
project-A/
project-B/
svn list file:///var/svn/multi-project-repo/project-A
trunk/
branches/
tags/
```
If the repository holds exactly one project, the repository root can
double as the project root (a bare `trunk/branches/tags` at the top).

## Hooks

A hook is an executable script the repository runs automatically when a
specific event happens. "Pre" hooks run before an operation and can reject
it (non-zero exit); "post" hooks run after and are read-only with respect
to the operation itself (useful for notifications, not enforcement).

Available hooks: `start-commit`, `pre-commit`, `post-commit`,
`pre-revprop-change`, `post-revprop-change`, `pre-lock`, `post-lock`,
`pre-unlock`, `post-unlock`.

Templates are dropped into `myrepo/hooks/` automatically at creation:
```bash
ls myrepo/hooks/
post-commit.tmpl  post-unlock.tmpl  pre-revprop-change.tmpl
post-lock.tmpl    pre-commit.tmpl   pre-unlock.tmpl
post-revprop-change.tmpl  start-commit.tmpl
```
To activate one on Unix: copy the `.tmpl` file to the same name **without**
the extension, edit it, and make it executable. On Windows, the file
extension itself must indicate an executable type (`.exe`, `.bat`, etc.) —
Windows doesn't use the Unix executable bit.

Critical gotchas:
- Hooks run with an **empty environment** — no `$PATH`, nothing. Use
  absolute paths to every program the hook calls, and set any environment
  variables the hook needs explicitly.
- Hooks execute as whatever OS user runs the server process (typically the
  service account for Apache/svnserve), so they need OS-level permission
  to do whatever they do, including reading the repository itself.
- **Never modify a commit's contents from a hook.** It's tempting to
  auto-fix style/policy violations in a `pre-commit` hook, but this
  desyncs the client's cached view of what it just committed from what's
  actually in the repository, causing confusing, hard-to-diagnose
  problems. Validate and reject non-compliant commits instead of silently
  rewriting them.

The most common use of `pre-revprop-change` specifically is enabling
revision-property edits at all — by default, editing an unversioned
revprop (e.g. fixing a typo in a commit message after the fact) is
disabled server-side until this hook exists and permits it, precisely
because revprop changes are not versioned and can't be undone.

## The svnadmin / svnlook toolkit

Both are server-side-only (local paths, not URLs):

- **`svnadmin`** — repository creation, maintenance, migration:
  `create`, `dump`/`load` (export/import for migration or backup),
  `hotcopy` (safe live backup), `verify`, `recover` (fix a wedged BDB
  repository), `pack` (FSFS space optimization), `setlog`/`setrevprop`
  (revprop edits from the server side, bypassing hooks), `lslocks`/
  `rmlocks`.
- **`svnlook`** — read-only inspection of a repository's current or
  pending (in-transaction) state: `svnlook log`, `svnlook changed`,
  `svnlook diff`, `svnlook tree`, `svnlook author`. Commonly called from
  inside hook scripts to inspect the commit that's in flight.
- **`svndumpfilter`** — include/exclude specific paths when replaying a
  dump file; the practical way to permanently strip something
  sensitive out of history (SVN has no "delete this from history" command
  otherwise, by design — see `branching-tagging-merging.md`'s note on
  undoing commits).
- **`svnsync`** — one-way mirroring of a repository (full or partial) to
  another location, revision property included.
- **`svnrdump`** — dump/load over the network for a *remote* repository
  you don't have filesystem access to (`svnadmin dump`/`load` require
  local access).

## Choosing a server: svnserve vs. SSH vs. Apache

No universally "best" option — trade-offs only:

**Plain `svnserve`** — fastest to set up, stateful protocol (faster than
WebDAV), no OS accounts needed, password never sent over the network.
Downside: traffic unencrypted by default and passwords stored in clear
text server-side unless you additionally configure SASL; minimal logging;
no built-in web browsing of the repo.

**`svnserve` over SSH** — reuses existing SSH accounts/infrastructure,
still fast (stateful protocol), fully encrypted. Downside: only one auth
method, minimal logging, requires shared system group or SSH key
management, easy to misconfigure file permissions.

**Apache + `mod_dav_svn`** — integrates with any Apache auth scheme, full
Apache logging, SSL, works through corporate firewalls (it's just HTTP/S),
built-in repository browsing, can be mounted as a WebDAV network drive.
Downside: noticeably slower than `svnserve` (stateless protocol, more
round-trips), more complex initial setup.

**Rule of thumb from the SVN Book's authors:** start with plain
`svnserve` for a small team getting going — least setup, fewest moving
parts — and move to Apache later only if you specifically need its
integration or web-browsing features. If you need to plug into existing
identity infrastructure (LDAP, Active Directory, X.509), you need either
Apache or `svnserve` with SASL. **Avoid exposing repositories directly via
`file://` to multiple users** (or the equivalent `svn+ssh://` local-account
pattern) — it removes every layer of access control between users and the
raw repository data.

Regardless of which you pick: run the server process as a single dedicated
`svn` OS user, and make that user the sole owner of the repository
directory.

Deliberately out of scope here: the actual configuration syntax once
you've picked one (`svnserve.conf` and `authz`/`passwd` file formats for
svnserve, or the Apache `httpd.conf`/`mod_dav_svn` directives and
`AuthzSVNAccessFile` setup for Apache). That's a bigger, more
install-specific topic than fits a "which one and why" overview — the SVN
Book's own Server Configuration chapter is the place for it if you need
the exact directives.

## Backup and disaster recovery basics

- **`svnadmin hotcopy SRC DST`** — safe, consistent live backup while the
  repository is in use.
- **`svnadmin dump`** / **`svnadmin load`** — portable, human-inspectable
  backup/migration format; also how you migrate between FSFS and BDB
  backends, or between major Subversion versions when a direct upgrade
  isn't supported.
- **`svnadmin verify`** — integrity check.
- **`svnadmin recover`** — fixes a BDB repository "wedged" by a crashed
  process holding a database lock (see the "three meanings of lock" note
  in `properties-and-metadata.md`). Not applicable to FSFS, which doesn't
  have this failure mode.
