# Discussion

The `discussion` role holds changes under review and what reviewers say
about them: Gerrit changes, GitHub pull requests, GitLab merge requests,
patch series on a mailing list.

Terms:

    CHANGE      one unit of review, by an id its ADAPTER defines: a Gerrit
                change number, a pull request number, the Message-Id of a
                patch
    REVISION    one version of a CHANGE: a Gerrit patch set, a pushed head
                commit, v3 of a mailed patch
    SERIES      CHANGEs that depend on each other, base to tip
    THREAD      comments that reply to each other, about one ANCHOR
    ANCHOR      what a THREAD is about: a CHANGE, its commit message, a
                file, or lines of a file
    VOTE        a reviewer's verdict on a REVISION: a Gerrit label, a GitHub
                approval, a Reviewed-by: trailer
    DRAFT       a comment only the human sees until they publish it

## Commands

    locate COMMIT       print the change RECORD whose REVISION is COMMIT
    change CHANGE       print the change RECORD
    series CHANGE       print the change RECORDs of CHANGE's SERIES, base to tip
    threads CHANGE      print the thread RECORDs of CHANGE
    drafts CHANGE       print the human's DRAFTs on CHANGE as thread RECORDs
    draft CHANGE        add a DRAFT, read as JSON from stdin; print its RECORD
    discard CHANGE ID   delete the DRAFT ID
    publish CHANGE      publish the DRAFTs on CHANGE, with a message and
                        VOTEs read as JSON from stdin

`publish` runs only on the human's explicit action.

## Capabilities

`capabilities` prints an object with these booleans:

    exact       ANCHORs are exact; without it, they are inferred, e.g.
                from a quoted hunk in a mail, and may be wrong
    resolved    the SOURCE marks THREADs resolved
    drafts      `drafts`, `draft`, `discard` and `publish` work
    series      `series` works
    votes       the SOURCE has VOTEs

```json
{"exact": true, "resolved": true, "drafts": true, "series": true, "votes": true}
```

## Change

`kind` is `change`. `data`:

    title       the subject line
    url         where a browser shows it, or null
    status      open, merged or abandoned
    owner       a PERSON
    revisions   REVISIONs, oldest first
    current     id of the newest REVISION
    votes       VOTEs on the newest REVISION
    message     the commit message of the newest REVISION, or the
                description of a pull request

`links` include `change` and, when the SOURCE has one, `change-id`: an
identity that survives rebases, such as a Gerrit Change-Id or a b4
change-id.

A REVISION:

    id          as the SOURCE numbers it: "5", "v3", a commit
    commit      the commit, or null when the SOURCE only has a mail
    parents     parent commits
    fetch       a ref to fetch it from, e.g. refs/changes/34/1234/5, or null
    uploader    a PERSON
    created     the time, RFC 3339

A VOTE:

    label       e.g. Code-Review, approved, Reviewed-by
    value       an integer; 1 for a trailer or an approval
    by          a PERSON

A PERSON:

    name        the display name
    email       or null
    username    or null
    me          true when this is the human running the ADAPTER

## Thread

`kind` is `thread`. `id` is the id of its first comment. `data`:

    anchor      an ANCHOR
    revision    id of the REVISION the THREAD started on
    resolved    true or false; null when the SOURCE has no such mark
    comments    COMMENTs, oldest first

A COMMENT:

    id          unique within the SOURCE
    author      a PERSON
    revision    id of the REVISION it was written on
    text        the text
    created     the time, RFC 3339
    reply_to    id of the COMMENT it replies to, or null
    draft       true for a DRAFT

An ANCHOR is one of:

    {"on": "change"}
    {"on": "message"}
    {"on": "file", "path": PATH}
    {"on": "lines", "path": PATH, "side": SIDE, "start": LINE, "end": LINE}

PATH is relative to the root of the repository. SIDE is `base`, the
REVISION's parent, or `revision`. LINE counts from 1; `end` is included.
`lines` may also have `start_col` and `end_col`, counting characters from
1, `end_col` excluded.

```json
{"id": "c9d8e7", "source": "https://review.example.org", "role": "discussion",
 "kind": "thread", "basis": "5:f00d42",
 "links": [["change", "1234"], ["change-id", "I3f2a9c"]],
 "data": {
   "anchor": {"on": "lines", "path": "src/file.c", "side": "revision",
              "start": 412, "end": 415},
   "revision": "3",
   "resolved": false,
   "comments": [
     {"id": "c9d8e7", "author": {"name": "Jane Reviewer", "email": null,
      "username": "jreviewer", "me": false}, "revision": "3",
      "text": "This leaks the lock on the error path.",
      "created": "2026-10-20T14:02:11Z", "reply_to": null, "draft": false}
   ]},
 "notes": {}}
```

## Draft input

`draft` reads one object:

    reply_to    id of the COMMENT to reply to; or
    anchor      an ANCHOR, to start a new THREAD
    revision    id of the REVISION, when starting a THREAD
    text        the text
    resolved    true or false: mark the THREAD; omitted, unchanged

`publish` reads one object:

    message     a message on the CHANGE, or null
    votes       an object of label to value, e.g. {"Code-Review": 1}

## Not in this role

Whose turn it is in a THREAD, whether an author is a bot, and whether a
later REVISION addressed a comment are judgements, not data. STEPs make
them and keep them in `notes`.
