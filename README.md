# PR Reactor - PRD

The PR Reactor is a developer-side service that tracks a developer's open PRs on any GitHub repo and runs user-defined hooks as each PR moves through its lifecycle, alongside CI, review bots and people.&#32;

## Problem and goals

Agentic development patterns increases the number of PR's a developer is managing. Often developers work with sophisticated adversarial review systems, CI workflows, security reviews, and other automated PR review systems that make manual "pr shepherding" untenable.&#32;

A simple solution may involve a developer running an AI agent session with cron prompts, where every check costs model tokens and the work dies with the session. The PR Reactor tracks PRs, handles routine actions and keeps state on its own, and spends model time only on events that need judgment.

**Goals**

- No cost while idle: the reactor spends nothing on models or agents when nothing has changed.
- A change on any in-scope PR is picked up within minutes.
- Hands-off PRs: routine work happens automatically, and anything that needs judgment goes to whatever tool or agent the user chose.
- Works with any GitHub repo, maintainers configure the system based on their existing repo conventions.&#32;
- After a restart, syncs to where each PR actually is, including changes made while it was off, without repeating actions or notifications.
- An API and a web page shows every PR's state, history and stats, with controls to manage the reactors behavior.
- Users decide which states exist, which events move a PR between them and which hooks run in response. A PR's state follows what actually happened, whether the reactor, CI, a bot or a person caused it. The shipped states and hooks are sensible defaults for any repo, and extending them is a primary use case.

**Non-goals**

- An org-wide bot or GitHub App. The reactor runs on the developer's machine, under their identity, for the PRs its targeting rules select.
- Replacing CI, the merge queue or review bots. The reactor reacts to them; it doesn't re-implement them.
- Writing new features from issues. The reactor drives existing PRs to approval.
- Multiple Reactor instances targeting a single PR. The system is designed to operate as a single instance.&#32;
