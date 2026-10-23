# Tracker

The `tracker` role holds issues: bugs, tasks and requests, their state
and what people say about them.

Terms:

    ISSUE       one issue, by an id its ADAPTER defines
    QUERY       a search in the SOURCE's own language

## Commands

    issue ISSUE     print the issue RECORD
    search          print the issue RECORDs matching a QUERY read from
                    stdin
    comment ISSUE   add a comment, its text read from stdin
    transition ISSUE STATUS
                    move ISSUE to STATUS

Trackers have no drafts. `comment` and `transition` run only on the
human's explicit action.

## Capabilities

`capabilities` prints an object with these booleans:

    search      `search` works
    comment     `comment` works
    transition  `transition` works

## Issue

`kind` is `issue`. `data`:

    title       the summary line
    type        as the SOURCE names it, or null
    status      as the SOURCE names it
    closed      true when no more work is expected
    priority    as the SOURCE names it, or null
    assignee    a PERSON, or null
    reporter    a PERSON
    created     the time, RFC 3339
    updated     the time, RFC 3339
    url         where a browser shows it
    text        the description
    comments    COMMENTs, oldest first
    related     related ISSUEs: a list of [TYPE, ISSUE], TYPE as the
                SOURCE names it

A PERSON is as in [discussion.md](discussion.md). A COMMENT:

    id          unique within the SOURCE
    author      a PERSON
    text        the text
    created     the time, RFC 3339

`links` include `issue` and every `change` and `commit` the SOURCE links
to the issue.
