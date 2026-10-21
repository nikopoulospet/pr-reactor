# PR Reactor - PRD

Oct 20, 2026 · @Peter

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

## Users and personas

hooksthrough

- **Developer (primary).** Owns a handful of open PRs at once and wants them driven to approval without babysitting CI, review bots and merge queues. Needs to see where each PR stands, step in when asked, and pause or take back a PR at any time.
- **Repo maintainer.** Writes and reviews the repo's configuration, custom states and hooks so every developer on the repo gets the same behavior. Needs a stable extension surface and a way to test hooks before they act.
- **Reviewer or approver.** Never runs the reactor but sees its pushes and replies on PRs. Needs replies that are attributable, predictable and only approve or merge where the user's configuration allows it.

## Key use cases

1. A developer opens a PR and walks away; the reactor requests review, handles findings and failures, and notifies them only when the PR is ready for approval or needs a decision.
2. A developer checks every PR's state, history and spend from one page or API call, and pauses a PR or reruns its last failed hook from there.
3. A maintainer adds a repo-specific state (say, `security_review`) and a hook that calls their own script or service, with no change to the reactor.
4. A maintainer requires CI, adversarial review, security review and design review to pass before a PR can enter human review. These checks run in any order or in parallel, and the PR moves on once the last one passes.
5. A maintainer swaps the hook behind a judgment call from one agent CLI to another, or to an internal API, by changing the repo's configuration.
6. A developer runs a new hook in dry-run on live PRs and compares its proposed transitions with what they would have done.

## Requirements

**Functional**

1. Users set targeting rules that decide which PRs are in scope: for example all open PRs, PRs assigned to a given user or bot (such as Dependabot), or PRs matching an author, label or branch pattern.
2. The reactor notices a change on any in-scope PR within minutes.
3. Users define the states, events and hooks, and which transitions the reactor's hooks may make. The shipped ones are defaults that can be overridden or disabled.
4. Every state transition controlled by the reactor's hooks, including approving, enqueueing and merging, happens only where the user's configuration allows it.
5. Every hook ends in a proposed transition, and the reactor rejects one the configuration doesn't allow.
6. Transitions caused outside the reactor (CI, review bots, merge queues, people) are picked up and the PR's state updated to match. The reactor never blocks or reverts them.
7. A transition can require a set of passed checks or reached states, met in any order. A PR's lifecycle isn't a single linear flow: several tracks can run in parallel and join at a gate.
8. A hook can be any script, program or web service. The reactor doesn't depend on a particular agent or model vendor.
9. Repo-specific behavior (merge path, review bot, CI logs, notifications) can be changed per repo, with GitHub's own behavior as the default.
10. A repo's configuration lives in the repo, so it is shared and reviewed like code.
11. After a restart, the reactor syncs each in-scope PR to its current state on GitHub, including changes made while it was off, and carries on from there. It never repeats an action or notification it already sent.
12. The reactor's state doesn't depend on any agent session; ending a session loses nothing.
13. A developer can see each PR's stage, next step, transition history, hook runs and spend from a web page, a CLI or an API, and the view stays current with no model or agent calls.
14. A developer can pause a PR to take it off the reactor and resume it later, or rerun its last failed hook.
15. A dry-run mode runs the rules and hooks but changes nothing on GitHub, and shows what would have been pushed and posted.
16. Every hook's input and output is logged for review.

hook's input and output&#32;

**Non-functional**

- Runs unattended. keeps running across restarts.&#32;
- Acts with as whatever identities the environment’s own configuration provides holds no secrets of its own, and never logs secrets.
- Idle cost: no model calls, and GitHub usage well inside the user's rate limits.

## KPIs

These measure the reactor itself, independent of any repo's configuration or hooks. Targets are proposals to confirm.

| KPI | Definition | Proposed target |
| --- | --- | --- |
| Capacity | In-scope PRs tracked at once with every other KPI still met | 200 PRs |
| Detection time | GitHub state change → reactor records the new state | p95 under 2 min |
| Reaction time | GitHub state change → matching hook starts | p95 under 3 min |
| Reactor overhead | Time the reactor adds per hook run, excluding the hook's own runtime | p95 under 5 s |
| Concurrent hooks | Hooks running in parallel without delaying others | 10 |
| Restart sync time | Restart → every in-scope PR synced to its current GitHub state | under 5 min at capacity |
| Missed changes | GitHub state changes never recorded | 0 |
| Duplicate actions | Actions or notifications repeated, including after a restart | 0 |
| Disallowed transitions applied | Hook transitions applied despite the configuration rejecting them | 0 |
| Idle model spend | Model or agent cost while no in-scope PR changes | $0 |

## Phases

Each phase ships something usable on its own, and the next one starts only after its gate. Phases are proven first on a pilot repo with a non-standard merge path, then on a plain GitHub repo.

1. **Observe.** Track in-scope PRs and show each PR's state and history through the API, the web page and the CLI. No actions yet. *Gate: over a week on the pilot repo, the tracked history matches what actually happened on its PRs, with no missed merge-path changes.*
2. **Act without the model.** Built-in hooks handle routine events: requesting reviews, notifications, retargeting stacked PRs, and rerunning known infra failures. Ships the default configuration and the pilot repo's configuration. *Gate: the two cron loops are turned off in agent sessions.** Ty*
3. **Judgment hooks in dry-run.** User-defined hooks run on events that need judgment and propose transitions, replies and commits, but nothing reaches GitHub. *Gate: the developer accepts most proposals unchanged, and no hard-rule violation gets past the reactor's checks.*
4. **Judgment hooks live.** Proposals that pass the reactor's checks are applied to GitHub, one repo at a time. *Gate: two weeks on the pilot repo within the spending caps, with no hard-rule violations.*
5. **Second repo.** Prove the default configuration on a repo that uses native GitHub merging, with no changes to the reactor itself.
