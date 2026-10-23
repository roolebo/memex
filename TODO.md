# TODO

- What `comment`, `transition` and `retest` print: tracker.md and ci.md
  don't say. Adapters print nothing now.

- `revision` null on a comment tied to no revision, e.g. a GitHub
  conversation comment. discussion.md requires one.

- `memex bindings` prints `tracker - ` with a trailing space for a role
  turned off.

- A missing adapter with a single binding exits 127, not 1.

- Asking an adapter which roles it serves, without a binding.
