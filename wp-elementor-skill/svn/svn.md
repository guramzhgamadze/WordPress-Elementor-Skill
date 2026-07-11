# Subversion (SVN)

Practical reference for working with Apache Subversion, distilled from
*Version Control with Subversion* by Ben Collins-Sussman, Brian W.
Fitzpatrick, and C. Michael Pilato (the canonical "SVN Book", licensed
under CC BY 2.0 — https://creativecommons.org/licenses/by/2.0/ —
source at https://svnbook.red-bean.com/) — specifically its 1.7 edition,
the version linked when this skill was built. Command behavior described
here is accurate for any modern client (1.14.x and similar as of this
writing) unless a note says otherwise; SVN's core workflow has changed
very little since 1.7, with the occasional exception called out inline
(e.g. `--reintegrate` becoming automatic in 1.8+). This file covers what
comes up in day-to-day use. The `references/` folder goes deeper on
branching/merging, properties, repository administration, and
troubleshooting — read the relevant one when the task needs it.

## Mental model

Get this part right and almost everything else about SVN stops being
surprising:

- **Repository vs. working copy.** The repository is the single source of
  truth, living on a server (or in a local directory it doesn't matter
  which). A working copy is a local checkout you actually edit. Nothing you
  do locally affects anyone else until you `commit`; nothing anyone else
  does affects you until you `update`. Push and pull are always separate
  actions — committing never pulls in others' changes, and updating never
  publishes yours.
- **Revisions are global, not per-file.** Every commit — no matter how many
  files it touches — creates exactly one new revision number for the
  *entire repository tree*. "Revision 5 of foo.c" really means "foo.c as it
  appears in the repository snapshot taken at revision 5"; foo.c may not
  have changed at all between revisions 4 and 5. This is the single biggest
  mental adjustment for people coming from per-file systems.
- **Working copies are often mixed-revision.** After you commit, only the
  files you touched get bumped to the new revision number; the rest of your
  working copy stays where it was. This is normal, not a bug — `svn status
  -v` shows you the mixture. It does mean some operations (merge targets,
  reintegration) require an up-to-date, single-revision working copy first.
- **Copy-modify-merge, not lock-modify-unlock.** By default SVN lets
  everyone edit their own working copy freely and reconciles changes at
  commit time (conflicts are rare and usually easy). True exclusive locking
  exists but is meant only for genuinely unmergeable files like binary
  images — see `references/properties-and-metadata.md`.
- **Branches and tags are just directories.** SVN has no branch object.
  A "branch" is a directory created with a cheap, constant-time server-side
  `svn copy`; it's a branch only because people agree to treat it that way.
  A "tag" is exactly the same mechanism, used for a copy nobody is supposed
  to commit to. This is why the trunk/branches/tags layout convention
  (below) exists — SVN itself enforces none of it.

## If you know Git

SVN predates Git and works differently in ways that trip people up:

| Git | SVN |
|---|---|
| `git clone` | `svn checkout` (`svn co`) |
| `git add` (stage) + `git commit` (local) | `svn add` only schedules; there's no local commit — `svn commit` goes straight to the shared repository |
| `git commit` then `git push` | `svn commit` (one step, always "pushes") |
| `git pull` / `git fetch` + merge | `svn update` (`svn up`) |
| `git branch` (cheap, local, instant) | `svn copy trunk branches/name` (cheap, but it's a *server round-trip commit*, not local and instant) |
| `git merge` | `svn merge` — but you must be in the *target* working copy and run it explicitly; there's no "current branch" |
| `git log` | `svn log` (no `--oneline` by default; use `-q`) |
| `git diff` (unstaged) | `svn diff` (compares working copy to the pristine checked-out copy) |
| SHA hashes | Sequential global revision numbers (r1, r2, r3…) |
| `.gitignore` | `svn:ignore` property (per-directory, not inherited to subdirectories) + the `global-ignores` runtime config option |
| `git stash` | No real equivalent; closest is `svn diff > patch.txt` then `svn revert`, or a changelist (see `references/properties-and-metadata.md`) |
| Cheap local experimentation, rewrite history freely | Nothing is ever truly deleted or rewritten; history is append-only |
| Distributed, no "the" server | Centralized — one authoritative repository |

The practical upshot: **commits in SVN are immediately public.** There's no
staging area and no local-only commit, so don't reach for `svn commit` the
way you'd reach for `git commit` — it's closer to `git push`. Use `svn
status` and `svn diff` (no network needed) as your staging-area substitute
to review before you commit.

## The daily work cycle

This is the loop you'll use constantly. It's always some ordering of:
update → change → review → fix mistakes → resolve conflicts → commit.

**Get a working copy:**
```bash
svn checkout https://example.com/repo/trunk my-project   # full history download
svn export https://example.com/repo/trunk my-project     # no .svn metadata, for a clean release bundle
```

**Bring your working copy up to date:**
```bash
svn update          # svn up — pulls latest changes, merges into your working copy
svn update -r 1729  # backdate to a specific revision (whole WC or a path)
```

**Make changes.** Just edit files normally. For structural changes, tell SVN
explicitly:
```bash
svn add newfile.php          # schedule for addition (recursive if it's a directory)
svn add --force .            # bulk-add everything not ignored, without re-adding what's tracked
svn delete oldfile.php       # schedule for deletion (deletes locally now, removed from repo on commit)
svn copy foo.php bar.php     # duplicate with history retained
svn move old.php new.php     # rename; same as copy + delete
svn mkdir newdir             # create + schedule for addition
```
None of these touch the repository until you commit.

**Review before committing** (all fully offline — no network needed):
```bash
svn status           # svn st — overview: ? untracked, A added, D deleted, M modified, C conflicted
svn status -v        # long form — also shows working revision per item (reveals mixed revisions)
svn status -u        # -u contacts the repo, flags items that are out of date (asterisk marker)
svn diff             # line-level unified diff of local changes
svn diff -r 3        # diff working copy against a specific revision
svn diff -r 2:3      # diff two repository revisions directly
```

**Fix mistakes:**
```bash
svn revert file.php      # discard local changes, restore pristine version
svn revert -R .          # recursive revert of an entire directory tree
```
`svn revert` undoes *any* scheduled operation — an accidental `add`, `delete`,
or edit — not just content changes.

**Resolve conflicts.** You'll see this during `svn update` when someone
else's changes overlap yours:
```
Conflict discovered in 'bar.c'.
Select: (p) postpone, (df) diff-full, (e) edit,
        (mc) mine-conflict, (tc) theirs-conflict,
        (s) show all options:
```
Fastest paths: `p` to postpone and deal with it after the update finishes, or
pick a whole-file winner immediately with `svn resolve --accept mine-full
file` / `--accept theirs-full file`. For everything in between (editing conflict
markers by hand, partial accepts, non-interactive updates), see
`references/troubleshooting.md`.

**Commit:**
```bash
svn commit -m "Fix login redirect bug"     # svn ci
svn commit -F commit-message.txt           # message from a file
```
If a commit is rejected as "out of date," someone beat you to it — run `svn
update`, resolve anything that conflicts, and commit again. SVN will never
silently overwrite someone else's work.

## Examining history

```bash
svn log                        # reverse-chronological log for the current path
svn log -r 5:19                # revisions 5 through 19, chronological
svn log -r 19:5                # same range, reverse order
svn log -v                     # -v also lists every changed path per revision
svn log --diff                 # append a unified diff to each log entry (1.7+)
svn info                       # URL, revision, last-changed author/date for a path -- the quickest single-item lookup
svn cat -r 12 file.php          # print a file as it existed at revision 12
svn cat -r 12 file.php > old.php   # ...and save it
svn blame file.php              # a.k.a. svn annotate / svn praise — line-by-line last-changed attribution
svn list https://example.com/repo/trunk    # directory listing without checking anything out
```
Gotcha: `svn log` with no arguments right after a commit often *won't* show
the commit you just made — the parent directory's working revision usually
lags behind the file you touched. Run `svn update` first, or pass `-r`
explicitly.

## Repository layout: trunk / branches / tags

SVN imposes no structure — this convention is universal purely because
practically every project and every SVN-based platform (including the
WordPress.org plugin and theme directories) expects it:

```
project-root/
├── trunk/       # the main line of development
├── branches/    # divergent lines of development
└── tags/        # named, frozen snapshots — nobody commits here
```

Getting an existing unversioned tree into a fresh repository:
```bash
svn import /path/to/local/project https://example.com/repo/project/trunk -m "Initial import"
```
Note `svn import` commits directly — it does **not** turn your local folder
into a working copy afterward. Check out a fresh working copy separately if
you want to keep working on it.

**Creating a branch or tag is the same operation** — a server-side `svn
copy`, which is a cheap, near-instant commit that doesn't duplicate any
data:
```bash
svn copy https://example.com/repo/trunk https://example.com/repo/branches/my-feature \
  -m "Start my-feature branch"

svn copy https://example.com/repo/trunk https://example.com/repo/tags/1.2.0 \
  -m "Tag release 1.2.0"
```
The only difference between a branch and a tag is social convention: a tag
is a copy everyone agrees not to commit to. If someone does commit to it, it
has effectively become a branch. Full depth on branching, syncing, and
merging is in `references/branching-tagging-merging.md` — read it before
doing anything beyond a simple tag, especially before a `--reintegrate`
merge.

## Properties, in brief

Properties are versioned metadata (name/value pairs) attached to files,
directories, or revisions — separate from file content. The ones worth
knowing immediately:

- `svn:ignore` — per-directory list of filename patterns to hide from `svn
  status` / skip on `svn add --force`. Does **not** cascade to
  subdirectories (unlike `.gitignore`). Set it with `svn propedit svn:ignore
  path/` (multi-line values need an editor, not `propset`).
- `svn:mime-type` / `svn:executable` / `svn:eol-style` — file portability
  properties; auto-detected on `add`/`import` but worth checking on binary
  assets and shell scripts.
- `svn:externals` — pulls another repository path into a subdirectory
  automatically for everyone who checks out. Useful for vendored
  dependencies, with real sharp edges (see reference file).
- `svn:needs-lock` — makes SVN mark a file read-only until someone runs `svn
  lock` on it; the practical way to signal "this file can't be merged,
  please take turns" for binary assets.

Full command syntax (`propset`, `propedit`, `propget`, `proplist`,
`propdel`, revision properties) and gotchas are in
`references/properties-and-metadata.md`.

## Repository administration, in brief

```bash
svnadmin create /var/svn/myrepo          # create a repository (FSFS backend by default)
```
Hooks (`pre-commit`, `post-commit`, `start-commit`, `pre-revprop-change`,
etc.) live as executable scripts in `myrepo/hooks/` — templates are dropped
there automatically at creation. Choosing between `svnserve`, `svnserve`
over SSH, and Apache/mod_dav_svn, plus `svnadmin`/`svnlook` toolkit basics
and backup, are in `references/repository-admin.md`.

## Common errors and what they mean

| Message | Meaning | Fix |
|---|---|---|
| `Commit failed... File '...' is out of date` | Someone else committed since your last update | `svn update`, resolve, retry commit |
| `E155011` | The exact code behind the "out of date" message above | Same fix — update, resolve, retry |
| `Aborting commit: '...' remains in conflict` | A conflicted file has leftover `.mine`/`.rOLDREV`/`.rNEWREV` files | Resolve the conflict with `svn resolve`, then commit |
| `E155015` | The exact code behind the "remains in conflict" message above | Same fix — resolve, then commit |
| `Cannot merge into mixed-revision working copy` | Your working copy has multiple revisions in it | `svn update` first, then merge |
| `svn: warning: W160035: Path '...' is already locked by user 'X'` | Someone holds an exclusive lock | Ask them, or `svn unlock --force` / `svn lock --force` to break/steal it (see properties reference) |

More detail, plus mixed-revision quirks and `svn cleanup`, is in
`references/troubleshooting.md`.

## Worked example: publishing to a WordPress.org-style plugin/theme SVN

WordPress.org's plugin and theme directories run on exactly the
trunk/branches/tags/assets convention described above, so the general
workflow applies directly:

```bash
svn co https://plugins.svn.wordpress.org/your-plugin-slug
cd your-plugin-slug
# copy release-ready files into trunk/ (make sure vendored/composer
# dependencies are included -- there's no build step run on the server)
svn add trunk/*
svn commit -m "Initial release"

# tag the release so it's what actually gets served to users
svn copy trunk tags/1.0.0
svn commit -m "Tag 1.0.0"
```
Two platform-specific gotchas that aren't in the SVN Book itself but matter
a lot here: the `Stable tag` field in `readme.txt` must exactly match the
tag folder you create, or the directory serves the wrong version; and the
`assets/` folder (screenshots, icons, banners) sits as a sibling of `trunk`
and `tags` at the repository root, not inside either of them.

## Reference files

- `references/branching-tagging-merging.md` — creating and syncing
  branches, moving a working copy between branches with `svn switch`,
  `svn merge` mechanics, `--reintegrate`, mergeinfo, cherry-picking,
  undoing a bad commit, resurrecting deleted files, common branching
  patterns (release branches, feature branches), when *not* to branch.
- `references/properties-and-metadata.md` — full property command syntax,
  automatic property setting, `svn:ignore` in depth, `svn:externals` formats
  and pitfalls, the locking feature end to end (`svn lock`/`unlock`,
  breaking/stealing locks, `svn:needs-lock`), changelists.
- `references/repository-admin.md` — creating and administering
  repositories, writing hooks, the `svnadmin`/`svnlook` toolkit, choosing
  `svnserve` vs. `svnserve`-over-SSH vs. Apache/`mod_dav_svn`, backup and
  replication basics. (Choosing a server is covered; writing that server's
  detailed config files is explicitly out of scope — see the note inside.)
- `references/troubleshooting.md` — full conflict-resolution walkthrough
  (interactive options, merging conflict markers by hand, non-interactive
  mode), mixed-revision working copies explained, tree conflicts, `svn
  cleanup` and recovering from interruptions, the three different meanings
  of "lock" in SVN.

Read the relevant reference file in full before attempting anything beyond
the basics covered above — each goes considerably deeper than this
overview and includes the exact command sequences and warnings from the
source material.
