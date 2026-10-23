# memex

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
adapters. [discussion.md](discussion.md) specifies the discussion role.

[1]: https://www.theatlantic.com/magazine/archive/1945/07/as-we-may-think/303881/
