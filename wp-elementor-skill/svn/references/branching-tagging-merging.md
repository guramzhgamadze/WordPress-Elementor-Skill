# Branching, Tagging, and Merging

Deep reference for SVN's branch/tag/merge model. Read `svn.md` first for
the mental model and the basic `svn copy` mechanics — this file covers
everything past creating a simple branch or tag.

## Table of contents
- Why branch at all
- Creating and working with a branch
- Moving a working copy between branches (svn switch)
- Basic merging (sync merges)
- Mergeinfo: what it is and how to read it
- Reintegrating a branch back to trunk
- Undoing a bad commit
- Resurrecting deleted files or directories
- Common branching patterns
- To branch or not to branch

## Why branch at all

The alternative to branching is working in isolation in a single working
copy for days or weeks without committing. That's risky (one lost laptop
loses everything), inflexible (hard to share work-in-progress for review),
and it makes the eventual merge back much harder the longer you wait. A
branch gives you a place to commit incrementally — safe, reviewable,
shareable — without disturbing trunk.

## Creating and working with a branch

A branch is a directory created with a server-side `svn copy`. Copying on
the server is a constant-time "cheap copy" — SVN doesn't duplicate any
data, it just adds a directory entry pointing at an existing tree — so
branch creation is fast regardless of project size:

```bash
svn copy https://example.com/repo/calc/trunk \
         https://example.com/repo/calc/branches/my-feature \
         -m "Creating a private branch of /calc/trunk."
```

Do this as a **remote** copy (URL to URL), not by copying a directory
inside a working copy — a working-copy-side copy has to physically
duplicate every file on local disk and is much slower.

Check out the branch to work on it:
```bash
svn checkout https://example.com/repo/calc/branches/my-feature
```
It behaves like any other working copy. Commits to it are invisible to
people working on trunk (and vice versa) until someone merges.

Because SVN has no native branch concept — only copies — running `svn log
-v` on a file inside your branch shows its full history, including the
copy event itself and everything that happened to the file on trunk before
the branch point. This is normal and useful for auditing where a branch
came from.

### Moving an existing working copy between branches

Checking out a fresh working copy per branch isn't the only option.
`svn switch` re-points a working copy you already have at a different URL,
downloading only what's different:
```bash
svn switch ^/calc/branches/my-feature
```
It's a superset of `svn update` — an update moves a working copy through
time (same URL, newer revision); a switch moves it through time *and*
space (different URL). Local edits survive the switch, which is a handy
escape hatch if you've been editing trunk and realize partway through that
the work belongs on a branch (or vice versa): switch, then commit. You can
also switch just part of a working copy — a single subdirectory or even
one file — without touching the rest.

Don't confuse plain `svn switch` with `svn switch --relocate` (deprecated
since 1.7 in favor of the separate `svn relocate` command): relocate is
for when the repository's own URL changed — same content, new address —
not for moving between branches.

## Basic merging (sync merges)

**Requires Subversion 1.5+ client and server** for automatic merge
tracking; strongly recommend 1.7+ on both ends (most merge-tracking bug
fixes are client-side, so an old server with a new client is fine).

While you work on a branch, trunk keeps moving. Periodically pull trunk's
changes into your branch to avoid drifting so far apart that the eventual
merge back is painful:

```bash
cd my-feature-branch      # must be a CLEAN working copy — svn status shows nothing pending
svn merge ^/calc/trunk
```
`^/` is shorthand (1.6+) for the repository root, so you don't have to
type the full URL. Output looks like:
```
--- Merging r345 through r356 into '.':
U    button.c
U    integer.c
--- Recording mergeinfo for merge of r345 through r356 into '.':
 U   .
```
The "Recording mergeinfo" lines are SVN silently updating the
`svn:mergeinfo` property on the target — this is how it remembers what's
already been merged so the *next* sync merge only pulls in what's new. Do
not hand-edit this property.

**Important 1.7+ restriction:** `svn merge` refuses to run against a
mixed-revision working copy (`Cannot merge into mixed-revision working
copy`). Run `svn update` first.

After merging, build/test, then commit like any other change:
```bash
svn commit -m "Merged latest trunk changes to my-feature."
```

### Merging without automatic tracking (pre-1.5 servers)

If your server predates 1.5, none of the above bookkeeping happens
automatically. You must track merged ranges yourself, typically by noting
them in commit log messages, and merge explicit revision ranges:
```bash
svn merge ^/trunk -r399:HEAD
```
where 399 is the revision trunk was at when the branch was created (or the
end of the last manual merge). This is painful — a strong reason to insist
on 1.5+ everywhere.

## Mergeinfo: what it is and how to read it

`svn:mergeinfo` is the property SVN uses to track which changesets have
already been replicated into a given path. Inspect it directly:
```bash
svn propget svn:mergeinfo .
/trunk:341-390
```
Or ask SVN to interpret it for you:
```bash
svn mergeinfo ^/calc/trunk              # revisions already merged from trunk
svn mergeinfo ^/calc/trunk --show-revs eligible   # revisions NOT yet merged (a preview)
```
`svn mergeinfo` needs a source URL and takes an optional target URL
(defaults to the current working directory).

**Mergeinfo inheritance:** a path with the property explicitly set has
*explicit mergeinfo*, which its children inherit unless a child has its own
explicit mergeinfo (which always wins outright — explicit mergeinfo never
blends with inherited mergeinfo). A `*` suffix on a revision number in
mergeinfo output (e.g. `758*`) marks it as only *partially* merged —
merging it again would still produce additional changes.

**Subtree merges:** merging directly into some child of a branch root
(rather than the root itself) creates "subtree mergeinfo." It's handled
automatically but can make `svn propget svn:mergeinfo --recursive` output
large and confusing on complex repositories; use the `-v`/`--verbose` flag
for a more readable breakdown.

**Preview a merge without applying it:**
```bash
svn merge ^/calc/trunk --dry-run
```
Shows the status codes a real merge would produce without touching the
working copy. If you don't like the result of a real (non-dry-run) merge
before committing, `svn revert . -R` undoes it completely and you can
retry with different options — the merge isn't final until you commit.

## Reintegrating a branch back to trunk

Once your feature is done:

1. Do one final sync merge (pull latest trunk into the branch), build,
   test, commit.
2. Get a clean, up-to-date, single-revision working copy of **trunk**
   (not the branch) — a fresh checkout is simplest.
3. From that trunk working copy:
```bash
svn merge --reintegrate ^/calc/branches/my-feature
# build, test, verify
svn commit -m "Merge my-feature back into trunk!"
```

`--reintegrate` is a special-purpose flag: instead of replaying a
contiguous range of revisions (like a normal sync merge), it diffs the
latest trunk tree against the latest branch tree and applies exactly that
difference. It's required specifically for merging a branch *back* to its
parent, and it's picky:

- The trunk working copy must have **no local edits and no mixed
  revisions** — this is enforced, not just recommended.
- It only accepts a small set of other options: `--accept`, `--dry-run`,
  `--diff3-cmd`, `--extensions`, `--quiet`.
- **A branch that has been `--reintegrate`d cannot be usefully reused** —
  it can't cleanly absorb further trunk changes or be reintegrated again.
  After a successful reintegration, either delete the branch and recreate
  it fresh from trunk if you need to keep working:
  ```bash
  svn delete ^/calc/branches/my-feature -m "Remove my-feature, reintegrated in r391."
  svn copy ^/calc/trunk ^/calc/branches/my-feature -m "Recreate my-feature from trunk@HEAD."
  ```
  or use the "keep it alive" technique described in the SVN Book's Advanced
  Merging section (merge trunk into the branch once more immediately after
  reintegration, before continuing further branch work).

**Version note:** everything above describes 1.5–1.7 behavior, where you
must pass `--reintegrate` explicitly (this skill's source material is the
1.7 book). Since 1.8, `svn merge` detects a reintegrate situation
automatically and the flag is deprecated — `svn merge
^/calc/branches/my-feature` (no flag) does the right thing on its own from
a trunk working copy. Given how old 1.7 now is, assume auto-detection
unless `svn --version` confirms an older client; passing `--reintegrate`
explicitly on a modern client is harmless but unnecessary.

Deleting a merged-in branch does not lose history — `svn log` on the
`branches/` URL still shows it, and it can be resurrected later (see
below) if needed.

## Undoing a bad commit

`svn merge` can apply a changeset *backward*, which is how you roll back a
specific revision without touching anything else:
```bash
svn merge -c -303 ^/calc/trunk    # reverse-apply r303
svn status                        # review
svn diff                          # confirm the change is actually gone
svn commit -m "Undoing change committed in r303."
```
`-c -303` is shorthand for `-r 303:302`. This doesn't erase r303 from
history — anyone checking out an old revision between 303 and your revert
commit still sees the bad change — it only removes it from `HEAD` going
forward. True history deletion isn't supported by design (SVN revisions
are immutable, append-only trees); the closest tool for permanently
scrubbing something (e.g. an accidentally committed secret) is
`svndumpfilter`, covered in `repository-admin.md`.

## Resurrecting deleted files or directories

Two-step process: find the coordinate (revision + path), then copy it back.
The `@807` syntax below is a *peg revision* — it tells SVN which object
you mean (in case the path `real.c` refers to something different at
different points in history, e.g. it was deleted and something unrelated
was later added at the same path), as opposed to a plain `-r` revision,
which says which point in *that* object's own history you want. For a
straightforward resurrection like this they'll usually agree, but the peg
is what pins down the right one if a path has a complicated past.

1. Find when it was deleted:
```bash
svn log -v parent-dir/    # look for a D (deleted) entry for the path you want back
```
If it was deleted in revision 808, the last good version is at 807.

2. Bring it back — two options depending on whether you want to keep the
   historical link:
```bash
# Preserves full history (shows as "added with history", a '+' in status):
svn copy ^/calc/trunk/real.c@807 ./real.c
svn commit -m "Resurrected real.c from revision 807."

# Fresh start, no historical link:
svn cat ^/calc/trunk/real.c@807 > ./real.c
svn add real.c
svn commit -m "Re-created real.c from revision 807."
```
This also works entirely server-side, without a working copy:
```bash
svn copy ^/calc/trunk/real.c@807 ^/calc/trunk/ -m "Resurrect real.c from revision 807."
```

Prefer `svn copy` (not `svn merge -c -REV` reverse-applied) when only one
file among several changed in the deleting revision needs to come back —
reverse-merging the whole revision would also undo the other unrelated
changes it contained.

## Common branching patterns

- **Release branches.** Cut a branch when a version is about to ship
  (`branches/1.2.x`), so trunk can keep moving forward on new features
  while the release branch only receives stabilization fixes. Tag specific
  point releases (`tags/1.2.0`, `tags/1.2.1`) off the release branch as
  they ship.
- **Feature branches.** Cut a branch for any change large or disruptive
  enough that committing it incrementally to trunk would break things for
  everyone else. Sync with trunk regularly; reintegrate when done.

## To branch or not to branch

Branching has real overhead (context-switching, eventual merge effort).
For small, quick changes that won't destabilize trunk, just commit to
trunk directly. Reach for a branch when the change is large, long-running,
experimental, or otherwise risky enough that isolating it is worth the
merge-back cost later.
