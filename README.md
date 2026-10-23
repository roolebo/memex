# memex

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
`.memex`, the user binds it in their own config, or memex detects it.
Tools are composed around the records the adapters produce.

[PROTOCOL.md](PROTOCOL.md) specifies the roles, bindings, records and
adapters. [discussion.md](discussion.md), [tracker.md](tracker.md) and
[ci.md](ci.md) specify the roles.

## Usage

In a working copy:

    memex bindings                  what each role is bound to
    memex ROLE COMMAND [ARGUMENTS]  run a role's command

The review threads of the change at HEAD:

    memex discussion locate "$(git rev-parse HEAD)"
    memex discussion threads CHANGE

[skills/memex](skills/memex/SKILL.md) teaches an agent the same.

## Integrations

Adapters:

- [memex-gerrit]: discussion in Gerrit
- [memex-github]: discussion in GitHub pull requests; tracker in Issues
  and ci in Actions are planned
- [memex-jira]: tracker in JIRA
- [memex-jenkins]: ci in Jenkins

Planned:

- [memex-maloo]: ci in Maloo

Editors:

- [vim-memex]: review threads in the quickfix list and after their lines

[memex-gerrit]: https://github.com/roolebo/memex-gerrit
[memex-github]: https://github.com/roolebo/memex-github
[memex-jira]: https://github.com/roolebo/memex-jira
[memex-jenkins]: https://github.com/roolebo/memex-jenkins
[memex-maloo]: https://github.com/roolebo/memex-maloo
[vim-memex]: https://github.com/roolebo/vim-memex

[1]: https://www.theatlantic.com/magazine/archive/1945/07/as-we-may-think/303881/
[2]: https://commons.wikimedia.org/wiki/File:Memex_at_Das_Netz_exhibition_of_Deutsches_Technikmuseum.jpg
[3]: https://creativecommons.org/licenses/by/4.0/
