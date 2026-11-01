---
title: "Are GitHub Actions still worth it in the age of agentic workflows?"
date: 2026-11-01T00:00:00+01:00
description: "Why I'd keep GitHub Actions for builds, tests, and deployments, and use agents to investigate failures rather than replace the pipeline."
summary: "An agent can investigate a failed test, but I'd still want the pipeline to run it. Here's where I think GitHub Actions and agentic workflows fit together."
draft: false
---

The appeal of agentic workflows is easy to see. We can describe what we want done and let an agent work out how to do it, rather than write every step ourselves.

Does that mean we should spend less time building GitHub Actions workflows? I think it depends on what we're trying to automate.

For a continuous integration and deployment (CI/CD) pipeline, we normally spell out the steps:

{{< workflow caption="A familiar CI/CD pipeline" >}}
📥|Check out|Bring the repository onto the runner.|step
📦|Restore|Fetch the dependencies.|step
🛠️|Build|Turn source code into an application.|step
🧪|Test|Check that it behaves as expected.|step
📤|Publish|Package the verified result.|step
🚀|Deploy|Ship it to the world.|step
{{< /workflow >}}

We decide when it runs, which checks are required, and what should happen if one fails.

With an agentic workflow, we still configure triggers, tools, and permissions. But for the task itself, we can ask for something like:

- Investigate why this pull request is failing.
- Review this infrastructure change and identify potential problems.
- Look at this issue, inspect the repository, and propose an implementation.

The agent can decide which logs to read, which files to inspect, and whether it needs more information before suggesting an answer.

That's useful work. It just isn't the same work as compiling an application or enforcing a deployment gate. For me, that's a reason to keep investing in GitHub Actions rather than hand the whole job to an agent.

## It isn't really YAML vs. AI

There's a problem with treating this as **GitHub Actions vs. agentic workflows**: [GitHub Agentic Workflows](https://github.github.io/gh-aw/) actually run inside GitHub Actions.

What we're choosing is whether to give the system a set of steps to follow or a task it needs to reason through.

Take the CI part of the pipeline. A pull request might go through these checks before its changes are ready to publish:

{{< workflow caption="From pull request to a verified artifact" >}}
📦|Restore dependencies|Make sure the project has what it needs to build.|step
🛠️|Compile the application|Catch build errors before they land.|step
🧪|Run unit tests|Verify the code still behaves correctly.|step
🧹|Run linting|Catch style and static-analysis issues.|step
✅|Validate the build|Apply the remaining project-specific checks.|step
📦|Produce an artifact|Save the exact output for the next stage.|step
{{< /workflow >}}

If those six steps are required, a model doesn't need to decide whether the tests are worth running. We've already made that decision.

The workflow should run them and report what happened, even if that means a failed CI run. The next job is to understand the failure.

## The tests failed. Now what?

Imagine a GitHub Actions workflow runs the tests and reports:

```text
Build: ✅ Passed
Tests: ❌ Failed
```

We know the tests failed. What we might not know is whether the pull request broke something, the test is flaky, or the runner had a problem.

There are a few things to look into:

- Which code change, if any, is responsible?
- Is the failure specific to this pull request?
- Has this happened before, and is there a related issue?
- Is there evidence of a flaky test or an environment problem?
- What is the safest fix, and what evidence supports it?

We could add scripts to the workflow to answer each of those questions. Fetch earlier runs, compare failures, inspect the diff, look up related issues. That's possible, but covering all the combinations can turn into a lot of code.

This is where I'd consider an agent. Give it the failed run, the diff, and access to the relevant repository context, then ask it to investigate. Its explanation still needs checking, but we don't have to script every possible investigation ourselves.

I'd describe the difference like this:

**A traditional Actions job:** Run these checks.

**An agentic workflow:** Find out why these checks failed, using the tools and access I've allowed.

There's no reason those jobs need to compete with each other.

## Why not use agents for everything?

It's tempting to let the agent handle the whole thing, including deciding which checks to run. But suppose our repository requires this on every pull request:

```shell
dotnet test
```

An agent can run that command. I just wouldn't make a required check depend on it choosing to do so.

“This only changes documentation, so I'll skip the tests” might be a reasonable suggestion in some repositories. If our policy says the tests always run, though, the workflow needs to enforce that regardless of what the agent thinks.

I'd rather keep that part boring and predictable.

The same goes for:

- Compilation, linting, and artifact creation
- Security scanning and policy checks
- Release and deployment steps
- Approval gates

Of course, a pipeline isn't immune to surprises. Dependencies change, external services fail, and tests can be flaky. The point is that the required checks and gates aren't left to a model's discretion.

A required test failure should fail the CI run. A blocked deployment should stay blocked. We don't need to consult a model to enforce those rules.

There's also not much point paying for model reasoning on every run when a shell command already answers the question.

## It's not just test failures

Security scans and deployments can leave us with questions too.

After a security scan, we could ask:

> The scanner found three issues. Which ones were introduced by this pull request? Gather the evidence an engineer needs to review them.

After a failed deployment:

> Check the deployment logs, recent changes, and documentation. What is the likely cause, and what should we check next?

The scanner still runs. The deployment gates still apply. The agent helps us understand the result rather than decide whether we can ignore it.

## How I'd put the two together

For the failed-test example, I'd keep the setup fairly simple:

{{< workflow caption="From a failed test to an informed decision" >}}
⚙️|Action|Build the application and run the required tests.|step
🚨|Signal|CI reports a test failure.|step
🤖|Agent|Investigate logs, the diff, and repository context.|agent
🧾|Evidence|Summarize the likely cause and recommend a next step.|agent
👩‍💻|Engineer|Review the evidence and decide what to do.|human
{{< /workflow >}}

If the engineer applies a proposed fix, it goes back through the same checks. The explanation might sound convincing and still be wrong.

That's why I want the agent's findings to point to specific logs, files, and changes. A recommendation is much easier to review when I can follow the evidence behind it.

## Where this could save development time

The part that interests me most is how much work goes into building an investigation with scripts.

Fetching an issue or a test result is straightforward. Working out whether it's relevant to the current failure is harder. Once the automation needs to compare several sources and explain what they mean, the branching logic can grow quickly.

An agent lets us approach that differently. We provide:

- A goal
- Context
- Tools
- Permissions
- Constraints

Then it can choose what to inspect next. We still need to check the quality of its findings, but we don't have to write a separate branch for every new kind of failure.

That doesn't mean a long workflow should automatically become an agentic one. We've spent years encoding decision trees into pipelines and scripts:

```text
if this:
    do that
else if this:
    check something
if something_else:
    run another thing
else:
    ...
```

A deployment matrix or a release policy can be complicated and still have clear rules. I'd keep those rules in code.

If each new failure needs another attempt to interpret unfamiliar logs, that's a different problem. That's where an agent might save us some scripting.

For me, it's less about the amount of YAML and more about whether we know what the next step should be.

## Less scripting doesn't mean less responsibility

A shorter workflow definition can hide quite a lot of work. We still have to decide what the agent can access, how to assess its answers, and what happens when it gets something wrong.

Before relying on it, I'd want answers to questions like:

- Which repositories, logs, and APIs can it read, and what is it allowed to change?
- Could an issue comment or a log entry trick it into using that access in a way we didn't intend?
- Which decisions need an engineer's approval?
- Can we trace its conclusions back to evidence and see what it did when an answer is wrong?
- What will it cost, and could simpler automation do the same job?

These aren't all new concerns, but an agent can take a different path on each run. Asking it to “be careful” isn't a permission boundary. Access limits and approval gates need to be enforced outside the prompt.

## Use AI to write the workflow, too

There's another option that can get lost in this discussion: use AI to help write a normal GitHub Actions workflow.

If I need a GitHub Actions workflow that builds a .NET application, executes tests, caches dependencies, and publishes an artifact, AI coding tools can help me draft it, explain unfamiliar syntax, and make focused updates.

Once that workflow has been reviewed, it can run repeatedly without involving a model each time. If the job is always “restore, build, test, publish,” that seems like a sensible use of AI to me.

We still own the workflow and need to maintain it. But using AI to write it doesn't mean we need AI inside every run.

## A simple decision framework

When choosing an approach, I'd start with: **Do I know the steps, or only the outcome?**

{{< workflow layout="decision" caption="Pick the tool that matches the uncertainty" >}}
✅|Known steps|Use deterministic automation: build, test, package, deploy.|step|Deterministic
🔎|Unknown path|Consider an agent to investigate, gather evidence, and suggest a fix.|agent|Agentic
🔁|A bit of both|Combine them: Action → Agent → Action or human decision.|hybrid|Hybrid
{{< /workflow >}}

I'd also ask whether the agent can get the evidence it needs and whether its output will be easy to check. Knowing the outcome doesn't help much if we can't tell whether it reached a useful answer.

## So, are GitHub Actions still worth it?

Yes. I'd keep investing in reusable workflows for builds, tests, packaging, deployments, and policy checks. Those jobs haven't gone away.

What I wouldn't do is keep adding scripts indefinitely to explain every possible failure. Where an investigation depends on the logs, the diff, and the history of a particular problem, I'd consider an agent.

I still want CI to tell me that a test failed. I'd just rather spend less time working out why.
