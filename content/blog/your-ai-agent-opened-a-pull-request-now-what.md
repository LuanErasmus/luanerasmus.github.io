---
title: "Your AI Agent Opened a Pull Request. Now What?"
date: 2026-11-03T05:33:30+00:00
description: "An email-validation fix walks into a deployment workflow. A practical look at reviewing agent-authored pull requests without getting distracted by a convincing description and green checks."
summary: "The agent fixed email validation. It also changed deployment. Before you review whether the code works, ask how those two things ended up in the same pull request."
draft: false
---

*The agent has done the typing. You still have to do the deciding.*

You give an agent a bug. A little later, a pull request appears: tidy description, sensible-looking code, green checks.

Lovely. Coffee still warm. Practically the future.

Then you open **Files changed**.

## How did we get to deployment?

The issue was straightforward:

> **Bug:** Email validation accepts addresses without a domain.

The PR is... less straightforward.

![An agent takes a small request to fix alice@ and returns with a towering stack of changed files, including deploy.yml. The reviewer asks, “We were fixing an email, right?”](/img/agent-pr-scope.svg)

There might be a perfectly good explanation. Maybe the fix needs a library, and the build needs adjusting to support it. Or maybe the agent found a small bug and returned with a renovation project.

Either way, I'd pause before reviewing the implementation. Line 42 of `user-service.ts` can be beautifully written and still have no business being in this PR.

**First ask why the change belongs. Then ask whether it's correct.**

That's useful advice for any PR. With an agent, I'd make it a deliberate first step rather than assume we've agreed on the scope.

## The neat description isn't the receipt

Back in the PR description, everything sounds reassuring:

![A polished PR description says “Fixed. Tested. Ready.” A reviewer asks for the receipt: a regression test that fails before the fix and passes after it.](/img/agent-pr-evidence.svg)

Excellent. Those are now things to check, not things we know.

For our email bug, I'd look for a test using an address like `alice@`. Does it fail on the old implementation and pass with the fix? I'd also look at a valid address, so rejecting *everything* doesn't accidentally qualify as a successful repair.

“Comprehensive coverage” is a bigger claim than “added a test.” A test that only checks `alice@example.com` might pass before and after the change. It proves something, just not the thing we asked for.

The description can help you navigate. It can't do the review for you.

## The most interesting file might be missing

Now imagine the validation fix looks good, but no test file changed.

The diff won't wave a flag saying, “Something should probably be here.” It only shows what someone touched.

I'd follow the changed behaviour beyond those files. Does the registration endpoint use this validator? Does a background import use a different one? If the fix only reaches one path, we might have solved the bug for half our users.

Documentation, error messages, and logging deserve the same consideration when they're affected. Not every PR needs all of them. We're reviewing a change, not collecting a complete set of file extensions.

The useful question is: **What would I expect to change if I'd done this myself?**

## Green checks don't choose the design

Suppose the tests are good and the pipeline is green. Great. Keep those checks.

But CI isn't going to tap you on the shoulder and say, “There's already an email validator three folders over.”

![Green CI checks lead to a fork: reuse the existing validator or add a duplicate helper and dependency. Both paths can pass CI; a reviewer still needs to choose.](/img/agent-pr-design.svg)

I'd search for how the codebase already handles this. If an existing helper does the job, why introduce another? If the project consistently reports validation errors one way, why invent a second convention?

Sometimes a new approach really is warranted. The review should make that choice explicit, not let it arrive disguised as a bug fix.

## There's more below the waterline

This is how I think about the diff:

![An iceberg with the code diff above the waterline, and scope, assumptions, trade-offs, and impact below it.](/img/agent-pr-iceberg.svg)

With another developer, a surprising change usually starts a conversation: “Why this approach?” Their answer gives us context to examine.

An agent can offer an explanation too. I'd use it to find the relevant code, constraints, and evidence, rather than treat a plausible answer as proof of what happened.

“The code looks reasonable” is a weak substitute for understanding why we're merging it.

That doesn't mean demanding a transcript of every step or distrusting every line. It means following the surprising parts far enough to make an informed decision.

## Before pressing Merge

I don't want an enormous special ceremony for agent PRs. I'd keep a small checklist handy:

- **Problem:** Does this fix the bug we actually reported?
- **Scope:** Can I explain why every changed file is here?
- **Gaps:** Is a caller, test, or other affected surface missing?
- **Evidence:** Do the tests support the description's claims?
- **Fit:** Does this use the project's existing patterns where appropriate?
- **Risk:** If we're wrong, who notices, and how hard is it to undo?

That last question sets the depth of the review. A wording correction and an authentication change don't deserve the same amount of investigation, regardless of who wrote them.

The exciting part of agents is how much implementation work can happen before we open the PR. The trap is letting that convenience turn review into a quick scan for anything obviously broken.

We don't need to make review heavier. We need to start it a little earlier: at the problem and the scope, not just the code.

And if our email fix really does need a deployment change? Fine. Let's understand why.

**The agent can open the PR. You're still the one pressing Merge.**
