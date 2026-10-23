---
name: memex
description: Read the review threads, CI results and issues of the change
  the user works on, through memex. Use when the user asks about review
  comments, reviewer feedback, CI failures or bugs of their change, or
  about the review comment at their cursor.
---

# memex

`memex --help` lists the commands. PROTOCOL.md, next to this file,
specifies the RECORDs they print; each role's file next to it, its
commands and fields.

## Review of the current change

    memex review

prints the change RECORD of HEAD, then its threads. The change's `id` is
CHANGE for the other commands, e.g. `memex discussion series CHANGE`.

Open threads:

    memex review | jq -c 'select(.data.resolved == false)'

Each thread's `data.anchor` says what it is about; its line numbers are
those of `data.revision`. When that isn't the change's `data.current`,
the lines may have moved: say so.

## The comment at the cursor

With an editor registered in hunch, take its file and line from
`hunch-editor context`, make the file relative to the repository root,
and keep the threads whose anchor covers it:

    memex discussion threads CHANGE | jq -c --arg path "$PATH_IN_REPO" \
      --argjson line "$LINE" 'select(.data.anchor.on == "lines"
        and .data.anchor.path == $path
        and .data.anchor.start <= $line and $line <= .data.anchor.end)'

## Drafts

Reply to a thread, the id being one of its comments':

    jq -nc --arg to ID --arg text "$TEXT" '{reply_to: $to, text: $text}' |
      memex discussion draft CHANGE

A new thread on lines of a revision:

    jq -nc --arg text "$TEXT" '{anchor: {on: "lines", path: PATH,
      side: "revision", start: 10, end: 12}, revision: "15", text: $text}' |
      memex discussion draft CHANGE

Add `resolved: true` to mark the thread done. Then:

    memex discussion drafts CHANGE        the user's drafts
    memex discussion discard CHANGE ID    delete one

## Writing

`publish`, `comment`, `transition` and `retest` reach other people. Run
them only when the user asks for that exact action. Write replies as
drafts and let the user publish them.
