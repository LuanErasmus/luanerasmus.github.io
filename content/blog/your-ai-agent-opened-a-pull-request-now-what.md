---
title: "Your AI Agent Opened a Pull Request. Now What?"
date: 2026-11-03T05:33:30+00:00
description: "How to review a large agent-authored pull request: check the scope, verify the evidence, request a smaller revision, and use the findings to improve the next development cycle."
summary: "The agent opened a PR with more changes than you expected. Here's how I'd work out what belongs, ask for a better revision, and carry the lessons into the next task."
draft: false
---

You give an agent a bug. A little later, a pull request appears: tidy description, sensible-looking code, green checks.

Lovely. Coffee still warm. Practically the future.

Then you open **Files changed**. There's rather more here than you expected.

![An agent takes a small request to fix alice@ and returns with a towering stack of changed files, including deploy.yml. The reviewer asks, “We were fixing an email, right?”](/img/agent-pr-scope.svg)

How do you work out which changes belong? And once you've untangled this PR, how do you avoid repeating the same review on the next one?

I'd treat the review as another turn in the development cycle. The first PR is a proposed solution. We can challenge its scope, send it back with specific feedback, and check the revision before merging.

## Start with the task, not the first file

Before reading the diff line by line, reopen the issue or original request. Write down what should behave differently and what must keep working.

For a fictional email-validation bug, that might be:

- Registration must reject `alice@`.
- A valid address such as `alice@example.com` must still work.
- Registration must keep its existing validation-error response.

Those aren't a complete email specification. They're a starting point for reviewing this particular fix.

Now compare that with the PR. Has the agent fixed the reported behaviour, or expanded the task into replacing validation across the application?

If the requirement itself is unclear, settle that before asking for another implementation. Otherwise we're reviewing against a moving target.

## Make every group of changes earn its place

Scan the file list and group related changes before diving into implementation details. I'd ask the agent for a short account of each group:

> Group the changed files by purpose. For each group, explain which requirement it supports, why it's needed, and what would break without it. Separate required changes from optional cleanup, and point to the relevant code or configuration.

Use the answer as a review map, not an approval. A convincing explanation still needs checking.

For our example, the working notes could look like this:

| Change | What I'd check | Possible decision |
| --- | --- | --- |
| Registration validator | Does this change the reported behaviour and preserve the error response? | Keep if correct |
| Regression tests | Do they expose the bug before the fix? | Keep or strengthen |
| New package and helper | Does an existing validator already cover this behaviour? | Reuse it if suitable |
| Runtime and deployment | Does the proposed package require the upgrade? Is that package necessary? | Challenge the dependency chain |
| Unrelated cleanup | Would we need this without the bug fix? | Remove or propose separately |

That deployment edit might genuinely be required by the new package. But the new package might not be required by the task. Follow the reasoning back to the requested outcome rather than stopping at "this file depends on that one."

A large file count isn't automatically a problem. Generated files, lockfiles, or a shared API change can legitimately touch a lot of paths. The aim is to account for the scope, not hit an arbitrary limit.

If unrelated work is mixed in, ask for it to be removed or proposed separately. Don't split tightly coupled changes just to make the PR look smaller.

## Look outside the changed files

Once the scope makes sense, follow the behaviour through the codebase.

For registration, I'd find the entry point, the validator it calls, and the error returned to the user. Then I'd search for other callers and existing validation helpers. Does the proposed fix duplicate something? Would changing a shared helper affect an import job or another endpoint?

Check the tests around those paths too. A missing caller or regression test won't appear in the changed-files list.

This is also where I'd judge whether the implementation fits the project: existing error handling, dependency choices, logging, and public interfaces. Reuse an existing helper only after checking its behaviour, not just its name.

## Verify the claims in the PR

Now take the description's claims one at a time. "Fixed and tested" should lead to something you can inspect or reproduce.

![A polished PR description says “Fixed. Tested. Ready.” A reviewer asks for the receipt: a regression test that fails before the fix and passes after it.](/img/agent-pr-evidence.svg)

For the email bug, a test that only accepts `alice@example.com` isn't enough. It may pass before and after the change.

For this example, the checks we need would look roughly like this. These are illustrative assertions, not a complete email-validation specification:

```javascript
expect(isValidEmail("alice@")).toBe(false);
expect(isValidEmail("alice@example.com")).toBe(true);
```

Run the first assertion against the old implementation: it should fail. Run it with the fix: it should pass. Keep the valid-address check too, so rejecting everything doesn't qualify as a repair.

At least one test should exercise the real registration path, including its error response. Tests of a helper alone won't prove that registration uses it correctly.

Ask for the commands that ran, their results, and anything that couldn't run. Check the CI jobs themselves: a green status doesn't tell you whether the relevant tests ran or whether a required check was skipped. Reproduce the important behaviour locally or in a suitable test environment, with depth proportionate to the risk.

If the PR changes deployment, permissions, or a migration, unit tests won't cover all of that. Review those changes on their own terms before accepting them.

## Send back a bounded revision, then review it

"Make this simpler" leaves the agent guessing. Give it the finding, the expected result, and the boundaries.

If you find a suitable shared validator, a review comment could be:

> The import path already uses a validator that rejects `alice@`. Check whether registration can use it while preserving the existing error response. If it can, remove the new package, extra helper, and runtime/deployment changes. Add a regression test through registration, keep the valid-address case, and rerun the affected tests. Leave unrelated cleanup out of this PR. If reuse doesn't work, show the conflicting behaviour before broadening the change.

When the revision arrives, inspect what's changed since your last review, then check the complete PR again. Did it remove something required by another caller? Did a test get weakened to make it pass? Is there new work outside the agreed scope?

Confirm that the relevant checks ran against the latest revision. The earlier green run doesn't validate code added afterward.

Before merging, I want to be able to explain why each group of changes is present, what evidence supports it, and what we'd do if the change causes problems. A wording fix and an authentication change don't need the same investigation.

## Use the review to improve the next task

Removing the deployment edit fixes this PR. It doesn't tell the agent how to approach the next bug.

Look at what caused the extra work. Was the original request vague? Did the agent miss an existing helper? Did it treat a dependency upgrade as permission to change deployment?

For a similar task, I'd make those boundaries explicit up front:

> Fix the registration email-validation bug and add a regression test. Search for existing validators and callers before adding a helper. Preserve the current error response. Don't add dependencies, upgrade the runtime, or change deployment without first explaining why the fix needs it and getting agreement. Leave unrelated cleanup for a separate proposal. Report the checks you ran and anything you couldn't verify.

For larger tasks, ask for a plan and likely affected areas before implementation. That gives you a chance to challenge the dependency or deployment decision before it spreads across the PR.

If the same misunderstanding keeps recurring, put the durable project convention in the repository's agent instructions, such as `AGENTS.md` or its existing equivalent. Record where shared validation lives, which changes need agreement, and how to run the relevant checks. Don't turn one bug's acceptance criteria into a rule for every task.

Those instructions won't guarantee a clean PR. They give the next cycle a better starting point, and the next review tells you whether they helped.

**The agent can open the PR. You're still the one pressing Merge.**
