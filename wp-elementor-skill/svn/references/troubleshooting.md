# Troubleshooting

Deep reference for conflict resolution, mixed-revision confusion, and
recovering a stuck working copy.

## Table of contents
- Full conflict-resolution walkthrough
- Merging conflict markers by hand
- Mixed-revision working copies explained
- Tree conflicts (structural conflicts)
- svn cleanup and recovering from interruptions
- Quick error-message lookup

## Full conflict-resolution walkthrough

Conflicts surface during `svn update` (or `svn merge`, or `svn switch`)
when incoming changes overlap your uncommitted local edits. SVN stops and
asks interactively:
```
Conflict discovered in 'bar.c'.
Select: (p) postpone, (df) diff-full, (e) edit,
        (mc) mine-conflict, (tc) theirs-conflict,
        (s) show all options:
```
Full option set (via `s`):

| Key | Action |
|---|---|
| `e` (edit) | Open the file with conflict markers in `$EDITOR` |
| `df` (diff-full) | Show all local changes plus the conflicting region |
| `r` (resolved) | After manually fixing a file, tell SVN it's resolved |
| `dc` (display-conflict) | Show only the conflicting regions, not the whole diff |
| `mc` (mine-conflict) | Your version wins *only* where there's an actual conflict; non-conflicting incoming changes still merge in |
| `tc` (theirs-conflict) | Their version wins only where conflicting; your non-conflicting changes are kept |
| `mf` (mine-full) | Discard the entire incoming update for this file; keep your whole file as-is |
| `tf` (theirs-full) | Discard all your local changes to this file; take the server's version entirely |
| `p` (postpone) | Leave it conflicted, deal with it after the update finishes |
| `l` (launch) | Open a configured external merge tool |
| `s` (show all) | Print this menu |

`mc`/`tc` vs `mf`/`tf` is the distinction that trips people up: the
`*-conflict` pair only overrides the specifically-overlapping lines and
still merges everything else cleanly; the `*-full` pair throws away one
side's changes to the file **entirely**, conflicting or not.

Postponing (`p`, or pre-committing to it with `svn update
--non-interactive`, which auto-marks every conflict `C` without prompting)
leaves three extra unversioned files next to the conflicted one:
- `filename.mine` — your working copy version right before the update
  (omitted if SVN considers the file unmergeable/binary)
- `filename.rOLDREV` — the file as it was at your last-synced (`BASE`)
  revision
- `filename.rNEWREV` — the version just received from the server

**SVN will refuse to commit a file with these leftover files present.**
Resolve first:
```bash
svn resolve --accept base file.c          # revert to the pre-edit checked-out version
svn resolve --accept mine-full file.c     # keep only your version
svn resolve --accept theirs-full file.c   # keep only the server's version
svn resolve --accept working file.c       # accept whatever you hand-edited the file to (see below)
```
`svn resolve` requires explicit filenames — it won't guess which
conflicted file(s) you mean.

## Merging conflict markers by hand

If you edited the file directly to resolve a conflict (option `e`, or just
opening it in your own editor), it'll contain markers like:
```
<<<<<<< .mine
Salami
Mortadella
=======
Sauerkraut
Grilled Chicken
>>>>>>> .r2
```
Everything between `<<<<<<< .mine` and `=======` is your version; between
`=======` and `>>>>>>> .rN` is the incoming version. Edit the file down to
what it *should* say, delete the marker lines entirely, save, then:
```bash
svn resolve --accept working file.c
svn commit -m "..."
```
Forgetting to remove the marker lines themselves is the classic mistake —
SVN will happily commit a file that still contains literal `<<<<<<<` text
once you've told it the conflict is resolved.

## Mixed-revision working copies explained

Normal, not a bug: after a commit, only the files/directories you actually
touched get bumped to the new revision number. The rest of the working
copy stays at whatever revision it was already at. Run `svn status -v` to
see the actual mixture — the second column is each item's individual
working revision.

Why this matters practically:
- You **cannot** delete a file/directory that isn't fully up to date —
  SVN blocks it to avoid destroying changes you haven't seen yet.
- You **cannot** commit a property change to an out-of-date directory, for
  the same reason.
- As of 1.7, **`svn merge` refuses to target a mixed-revision working
  copy** by default (`--allow-mixed-revisions` overrides this, but only do
  so if you understand the consequences — merges into mixed-revision
  copies can produce spurious conflicts).
- `svn log` on a working-copy path can show a truncated or seemingly
  "wrong" history if that path's local working revision is older than you
  expect — it reports history *as of the working revision*, not `HEAD`.

Mixed revisions are also genuinely useful — deliberately backdating part
of a working copy to test an older snapshot of a subdirectory, for
instance — so the goal isn't to avoid them entirely, just to know a clean
`svn update` at the top of the working copy is what gets you back to a
single, uniform revision when an operation requires one.

## Tree conflicts (structural conflicts)

Distinct from content/text conflicts: a tree conflict happens when an
update or merge can't reconcile a *structural* change — e.g. someone
deleted or moved a file on the server while you also had it locally
modified, or renamed, or deleted. SVN can't guess intent here (was the
file supposed to still exist or not?), so it flags a tree conflict rather
than guessing.

```bash
svn status      # tree-conflicted items show a C in status too
svn info file   # shows a description of the specific tree conflict
```
Resolution generally means deciding by hand which side's structural intent
should win (keep your local move, or accept the server's deletion, etc.),
then `svn resolve` once the working copy reflects your decision. There's
less blanket "accept mine/theirs" automation here than for text conflicts
because the possible structural combinations vary so much — check `svn
info` on the conflicted path for specifics before deciding.

## svn cleanup and recovering from interruptions

If an `svn` command gets interrupted (killed, crashes, loses network
mid-operation), the working copy can be left administratively locked —
shown as an `L` in `svn status` (this is a *working-copy* lock, one of
three unrelated things SVN calls a "lock"; see
`properties-and-metadata.md` for the other two). Clear it with:
```bash
svn cleanup
```
Run this from the top of the affected working copy. It also removes any
leftover temporary files from the interrupted operation. If a large
operation keeps getting interrupted before it can finish, SVN 1.7+
generally allows safely resuming — re-run the same command rather than
starting over.

## Quick error-message lookup

| Message (paraphrased) | What's actually going on |
|---|---|
| `Commit failed... File '...' is out of date` | Someone else committed to that file since your last update |
| `svn: E155011` | The precise code behind that "out of date" message |
| `Aborting commit: '...' remains in conflict` | Leftover `.mine`/`.rOLDREV`/`.rNEWREV` files — resolve before committing |
| `svn: E155015` | The precise code behind that "remains in conflict" message |
| `Cannot merge into mixed-revision working copy` | Run `svn update` first |
| `svn: E195020` | The precise code behind that mixed-revision merge block |
| `svn: warning: W160035: Path '...' is already locked by user 'X'` | Someone holds an exclusive `svn lock`; ask them or use `--force` to break/steal it |
| `The subversion command line tools are no longer provided by Xcode` | macOS-specific — Apple dropped `svn` from Xcode Command Line Tools years ago; install via Homebrew (`brew install subversion`) instead |
| `svn: E155036` (working copy too old) | Working copy format predates the client version; run `svn upgrade` in it |
