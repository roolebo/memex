# Protocol

A project keeps its data in sources. Adapters read a source and print
records. Steps process records in a pipeline. Consumers, such as an editor
or an agent, show records or act on them. Only adapters talk to a source.

Terms:

    SOURCE      a place a project keeps data in, e.g. a Gerrit server, a
                mailing list archive, a JIRA instance
    ROLE        what a source holds for the project: code, discussion,
                tracker or ci
    ADAPTER     a program that implements one or more ROLEs for one kind
                of SOURCE, e.g. memex-gerrit for discussion in Gerrit
    ADDR        where an ADAPTER reaches its SOURCE, in a form the ADAPTER
                defines, e.g. https://review.example.org
    BINDING     a ROLE of a project bound to an ADAPTER and ADDR
    RECORD      one JSON object an ADAPTER or a STEP prints
    STEP        a program that reads RECORDs on stdin and prints RECORDs
    REPO        a working copy, by its absolute path

## Roles

- `code`: the repository, its workspaces and what the server has.
  Specified in [code.md](code.md).
- `discussion`: changes under review, their revisions, threads and votes.
  Specified in [discussion.md](discussion.md).
- `tracker`: issues, their state and comments. Specified in
  [tracker.md](tracker.md).
- `ci`: test runs, their results and logs. Specified in [ci.md](ci.md).

One adapter serves every role its service has: memex-github serves
discussion, tracker and ci. A project may use only some of them.

## Bindings

A project has at most one binding per role, except `discussion` and `ci`,
which may have several: a project may review both in Gerrit and on a
mailing list, and build in one CI system and test in another.

A project keeps its bindings in `.memex`, under version control, so
whoever clones it has them. A user may override them. For each role, the
first of these that binds it wins:

1. `$GIT_DIR/memex/bindings` in REPO: the user's, this working copy only.
2. `${XDG_CONFIG_HOME:-~/.config}/memex/bindings`: the user's, every
   working copy with a matching remote.
3. `.memex` at the root of REPO: the project's.

Nothing is guessed: a role no file binds has no adapter. ADAPTER `-`
turns a role off, e.g. a user's `tracker -` over the project's tracker.

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

    ADAPTER ADDR ROLE COMMAND [ARGUMENTS]

For a ROLE it doesn't implement, an adapter exits 2.

Every adapter implements, for each of its roles:

    capabilities    print one JSON object: what it supports in ROLE

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

## History

`memex history [CHANGE]` joins the `discussion` and `code` roles: every
version of CHANGE, or of HEAD's change, newest first. A version RECORD
has `source` `memex`, `role` `history` and `kind` `version`. `data`:

    revision    the REVISION's id, or null when only in the code
    commit      the COMMIT
    uploaded    true when the review has it
    current     true for the change's current REVISION
    where       `server` when uploaded, then the places the code role
                found it: WORKSPACEs, branches, upstream

## Writing back

An adapter writes to its SOURCE only as drafts: text only the human sees
until they publish it. Publishing runs only on the human's explicit
action, never on an agent's own decision.
