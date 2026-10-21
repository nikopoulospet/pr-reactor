# PR Reactor

## Goals

- No cost while idle: the reactor spends nothing on models or agents when nothing has changed.
- A change on any in-scope PR is picked up within minutes.
- Hands-off PRs: routine work happens automatically, and anything that needs judgment goes to whatever tool or agent the user chose.
- Works with any GitHub repo, maintainers configure the system based on their existing repo conventions.
- After a restart, syncs to where each PR actually is, including changes made while it was off, without repeating actions or notifications.
- An API and a web page shows every PR's state, history and stats, with controls to manage the reactors behavior.
- Users decide which states exist, which events move a PR between them and which hooks run in response. A PR's state follows what actually happened, whether the reactor, CI, a bot or a person caused it. The shipped states and hooks are sensible defaults for any repo, and extending them is a primary use case.

The full PRD lives in the [V1 epic](https://github.com/nikopoulospet/pr-reactor/issues/1).
