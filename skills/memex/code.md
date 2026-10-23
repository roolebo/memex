# Code

The `code` role holds the repository: commits, the workspaces the user
keeps of it, and what the server has that they lack. Steps use it to tell
whether a concern was addressed: in a later version, a later change of a
series, upstream, or only in a local workspace.

A logical change and the commits that record it are apart. Rewriting a
change (amend, rebase) makes a new commit; the change stays the same. The
version control or the review keeps an IDENTITY for it that survives
rewrites: a Gerrit Change-Id, a Jujutsu change id, a Perforce changelist.
Some identities name a whole series instead: a b4 change-id.

Terms:

    COMMIT      one recorded version, by the version control's id: a git
                or Mercurial hash, a Perforce changelist, an SVN revision
    IDENTITY    what names a logical change across its COMMITs: [SCOPE,
                VALUE], SCOPE being `commit` or `series`
    RANGE       COMMITs in the version control's notation, e.g.
                origin/master..topic in git
    WORKSPACE   a working copy of REPO on this machine, by its absolute
                path: a git worktree, a Mercurial share, a Perforce
                client, a Jujutsu workspace

ADDR is the server REPO is fetched from, e.g. its remote's URL. Local
commands read REPO and its other WORKSPACEs; `fetch` reaches the server.

## Commands

    workspaces          print the workspace RECORDs of REPO
    commits RANGE       print the commit RECORDs of RANGE, newest first
    versions VALUE      print the commit RECORDs, in any WORKSPACE, in
                        upstream and on the server when `remote`, whose
                        IDENTITY has VALUE
    diff COMMIT         print a file RECORD per file COMMIT changes
    compare OLD NEW     print the pair RECORDs of two versions of a series,
                        OLD and NEW being RANGEs
    base RANGE          print the commit RECORD where RANGE forks from
                        upstream
    fetch WHAT          fetch a version the server has and REPO lacks, e.g.
                        a REVISION's `fetch` from discussion.md; print its
                        commit RECORD

## Capabilities

`capabilities` prints an object with these booleans:

    local       the local commands work
    remote      `fetch` works, and `versions` asks the server
    exact       `compare` pairs COMMITs by IDENTITY or by what the version
                control records as their predecessor; without it, by
                matching their diffs, and pairs may be wrong

## Workspace

`kind` is `workspace`. `data`:

    path        the WORKSPACE
    branch      what it has checked out by name, or null
    head        the COMMIT it is on
    upstream    what it tracks, or null
    changed     true when it has changes not yet committed

## Commit

`kind` is `commit`. `data`:

    commit      the COMMIT
    identity    its IDENTITY, or null
    parents     parent COMMITs
    author      a PERSON, as in discussion.md, without `username`
    authored    the time, RFC 3339
    committed   the time, RFC 3339
    subject     the first line of the message
    message     the whole message
    trailers    the message's trailers, a list of [KEY, VALUE], when the
                version control has them; else empty
    where       where it was found: WORKSPACE paths, branches, upstream,
                the server

`links` include `commit` and, with an IDENTITY, `change-id`.

## File

`kind` is `file`. `data`:

    commit      the COMMIT
    path        the path after the change, relative to the repository root
    old_path    the path before, or null
    status      added, modified, deleted or renamed
    patch       the unified diff of the file

## Pair

`kind` is `pair`: a COMMIT of OLD matched with one of NEW. `data`:

    old         the COMMIT in OLD, or null when NEW added it
    new         the COMMIT in NEW, or null when NEW dropped it
    status      same, changed, added or removed
    patch       for changed, the diff between the two diffs

## Not in this role

Whether a commit addresses a review comment is a judgement, not data: the
analysis built on memex makes it, from diffs and pairs.
