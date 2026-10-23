# Protocol

A project keeps its data in sources. Adapters read a source and print
records. Steps process records in a pipeline. Consumers, such as an editor
or an agent, show records or act on them. Only adapters talk to a source.

Terms:

    SOURCE      a place a project keeps data in, e.g. a Gerrit server, a
                mailing list archive, a JIRA instance
    ROLE        what a source holds for the project: code, discussion,
                tracker or ci
    ADAPTER     a program that implements one ROLE for one kind of SOURCE,
                e.g. memex-gerrit for discussion in Gerrit
    ADDR        where an ADAPTER reaches its SOURCE, in a form the ADAPTER
                defines, e.g. https://review.example.org
    BINDING     a ROLE of a project bound to an ADAPTER and ADDR
    RECORD      one JSON object an ADAPTER or a STEP prints
    STEP        a program that reads RECORDs on stdin and prints RECORDs
    REPO        a working copy, by its absolute path

## Roles

- `code`: the repository, its branches and commits.
- `discussion`: changes under review, their revisions, threads and votes.
  Specified in [discussion.md](discussion.md).
- `tracker`: issues, their state and comments.
- `ci`: test runs, their results and logs.

One service may serve several roles; each role has its own adapter. GitHub
serving code, discussion and tracker for a project takes three adapters.

## Bindings

A project has at most one binding per role, except `discussion`, which may
have several: a project may review both in Gerrit and on a mailing list.

A project may keep its bindings in `.memex`, but needs none: the user's
bindings and detection work without it. For each role, the first of these
that binds it wins:

1. `$GIT_DIR/memex/bindings` in REPO: the user's, this working copy only.
2. `${XDG_CONFIG_HOME:-~/.config}/memex/bindings`: the user's, every
   working copy with a matching remote.
3. `.memex` at the root of REPO: the project's, under version control.
4. Detection: each installed adapter for the role is asked with `detect`,
   which looks at what REPO already has, such as its remotes or a
   `.gitreview`.

No adapter or ADDR is built in; they come only from these.

The files have one binding per line, fields separated by blanks. Lines
starting with `#` are comments. In `$GIT_DIR/memex/bindings` and `.memex`
the fields are ROLE, ADAPTER, ADDR:

    discussion  memex-gerrit   https://review.example.org
    discussion  memex-lore     https://lore.kernel.org/example
    tracker     memex-jira     https://jira.example.org
    ci          memex-jenkins  https://ci.example.org

In `~/.config/memex/bindings` a REMOTE comes first: the URL of one of
REPO's remotes.

    https://git.example.org/project.git  tracker  memex-jira  https://jira.example.org

## Adapters

An adapter is run as:

    ADAPTER detect REPO
    ADAPTER ADDR COMMAND [ARGUMENTS]

`detect` prints a line `ROLE ADDR` for each role whose SOURCE it
recognizes in REPO, and nothing when it recognizes none.

Every adapter implements:

    capabilities    print one JSON object: what this adapter supports

A role's specification lists its other commands and capabilities.

Commands print RECORDs on stdout, one per line. Input that isn't a short
argument, such as text, is read from stdin. An adapter never asks the
human anything.

Exit status:

- 0: done
- 1: failed, the reason is on stderr
- 2: not supported by this adapter or SOURCE

## Records

A RECORD has these fields:

    id          unique within its source
    source      the ADDR of the SOURCE, or the STEP's name for a RECORD
                a STEP creates
    role        the ROLE, or `note` for a RECORD a STEP creates
    kind        what it is within the ROLE, e.g. `thread`
    basis       a string that changes whenever the content changes; the
                ADAPTER chooses it
    links       keys other RECORDs can be joined on: a list of [TYPE,
                VALUE], e.g. ["change-id", "I3f2a..."], ["issue",
                "BUG-123"], ["message-id", "<...>"], ["commit", "4a2f..."]
    data        the content, as the ROLE specifies for this kind
    notes       what STEPs added: an object with one key per STEP name

A `note` RECORD also has:

    about       ids of the RECORDs it is about

```json
{"id": "a1b2c3", "source": "https://review.example.org", "role": "discussion",
 "kind": "thread", "basis": "ps5:c9d8e7",
 "links": [["change", "1234"], ["change-id", "I3f2a9c"]],
 "data": {"...": "..."},
 "notes": {"triage": {"attention": "reply"}}}
```

Rules for STEPs:

- A STEP writes only its own key in `notes`. It passes other keys, and
  every field it doesn't own, unchanged.
- A STEP passes RECORDs it doesn't process unchanged.
- A STEP that stores results keeps the `basis` they were computed from. A
  different `basis` makes them stale.

## Writing back

An adapter writes to its SOURCE only as drafts: text only the human sees
until they publish it. Publishing runs only on the human's explicit
action, never on an agent's own decision.
