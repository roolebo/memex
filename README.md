# memex: Work as you think

<img src="memex.jpg" alt="A reconstruction of the Memex desk" width="400">

<sub>A Memex reconstruction at the Deutsches Technikmuseum, Berlin. Photo:
[Bunyk][2], [CC BY 4.0][3], resized.</sub>

memex brings a project's code, review discussions, bugs and CI results
into one format, so that small tools can process them in a pipeline and an
editor or an agent can show them. The name comes from Vannevar Bush's
Memex [\[1\]][1]: a desk where all of one's records meet, linked by
associative trails.

Each project keeps these in its own combination of places: one reviews in
Gerrit and tracks bugs in JIRA, QEMU hosts code and issues on GitLab
and reviews on a mailing list, Linux reviews by mail. memex doesn't
assume any combination. A project binds each role to an adapter in its
`.memex`, and a user may override that in their own config. Tools are
composed around the records the adapters produce.

[PROTOCOL.md][protocol] specifies the bindings, records and adapters.

## Roles

| Role       | What it holds                             | Spec            |
| ---------- | ----------------------------------------- | --------------- |
| discussion | changes under review, threads, votes      | [discussion.md] |
| tracker    | issues, their status and comments         | [tracker.md]    |
| ci         | builds and test runs, results, logs       | [ci.md]         |
| code       | commits, workspaces, the server           | [code.md]       |

## Usage

In a working copy:

    memex bindings                  what each role is bound to
    memex history [CHANGE]          every version of a change, uploaded or
                                    local, newest first
    memex ROLE COMMAND [ARGUMENTS]  run a role's command

For the change at HEAD:

    memex discussion locate "$(git rev-parse HEAD)"   the review
    memex discussion threads CHANGE                   its threads
    memex ci runs "$(git rev-parse HEAD)"             its builds and tests
    memex tracker issue ISSUE                         an issue it fixes

[protocol]: skills/memex/PROTOCOL.md
[discussion.md]: skills/memex/discussion.md
[tracker.md]: skills/memex/tracker.md
[ci.md]: skills/memex/ci.md
[code.md]: skills/memex/code.md

[skills/memex](skills/memex/SKILL.md) teaches an agent the same.

Tests run offline, with fake adapters: `test/run [TEST...]`.

## Integrations

What memex can do with each service:

| Reviews                                       | Gerrit | GitHub |
| --------------------------------------------- | :----: | :----: |
| Find the review of a commit, read its threads |   ✓    |   ✓    |
| Follow a patch series                         |   ✓    |        |
| Reply in drafts, then publish                 |   ✓    |        |

| Issues                 | JIRA | GitHub |
| ---------------------- | :--: | :----: |
| Read and search issues |  ✓   |   ✓    |
| Comment, change status |  ✓   |   ✓    |

| CI                                      | Jenkins | Maloo | Actions |
| --------------------------------------- | :-----: | :---: | :-----: |
| Find the runs of a commit, their suites |    ✓    |   ✓   |    ✓    |
| Results of each test                    |    ✓    |   ✓   |         |
| Logs                                    |    ✓    |       |    ✓    |
| Run again                               |    ✓    |   ✓   |    ✓    |

Adapters: [memex-gerrit], [memex-github], [memex-jira], [memex-jenkins],
[memex-maloo].

Editors:

- [vim-memex]: review threads in the quickfix list and after their lines

## Related

- [hunch]: the human, their editor and agents working together
- [withkeys]: API keys from pass for the commands that need them

[hunch]: https://github.com/roolebo/hunch
[withkeys]: https://github.com/roolebo/withkeys

[memex-gerrit]: https://github.com/roolebo/memex-gerrit
[memex-github]: https://github.com/roolebo/memex-github
[memex-jira]: https://github.com/roolebo/memex-jira
[memex-jenkins]: https://github.com/roolebo/memex-jenkins
[memex-maloo]: https://github.com/roolebo/memex-maloo
[vim-memex]: https://github.com/roolebo/vim-memex

[1]: https://www.theatlantic.com/magazine/archive/1945/07/as-we-may-think/303881/
[2]: https://commons.wikimedia.org/wiki/File:Memex_at_Das_Netz_exhibition_of_Deutsches_Technikmuseum.jpg
[3]: https://creativecommons.org/licenses/by/4.0/
