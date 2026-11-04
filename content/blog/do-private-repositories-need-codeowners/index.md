---
title: "Do our private repositories need a CODEOWNERS file?"
date: 2026-11-04T00:00:00+00:00
description: "How I'd use CODEOWNERS to help teams find the right reviewers for private repositories, and what still needs agreeing outside the file."
summary: "A private repository can have plenty of contributors and still leave people asking who owns it. Here's where I'd use CODEOWNERS, and where it won't help."
draft: false
---

A pull request changes `/infra/`. The author posts it in chat: "Who should review this?"

That's a problem `CODEOWNERS` can help with. It won't tell us who maintains the service or answers an incident, but it can save contributors from hunting for a reviewer.

The repository being private doesn't change that need. It does mean checking whether your GitHub plan supports the feature: private organization repositories need **GitHub Team or GitHub Enterprise**. See GitHub's [current availability](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners).

## Start with the review we want

Take `invoice-orbit`, a fictional billing service. The application team maintains it, while the platform team reviews infrastructure changes. Both teams have agreed to those responsibilities.

Here's `.github/CODEOWNERS`. The organization and team names are placeholders:

```text
# Default review ownership for the repository.
*                       @example-org/application

# Specialist review ownership for infrastructure.
/infra/                 @example-org/platform

# Review changes to the ownership rules themselves.
/.github/CODEOWNERS      @example-org/application
```

Each line pairs a file pattern with users or teams. When a non-draft pull request changes matching files, GitHub automatically requests their review. Draft PRs trigger those requests when marked ready for review.

The teams must be visible and have explicit write access to the repository. Individual members having access through another route isn't enough.

GitHub reads this file from the PR's **base branch**. For requests on PRs targeting `main`, merge the file into `main` first.

{{< workflow caption="A ready-for-review PR changes /infra/" layout="decision" >}}
💬|Ask in chat|Without rules, the author finds the team and requests review.|step
📬|Route by path|With CODEOWNERS, GitHub requests platform for /infra/.|step
{{< /workflow >}}

**A review request doesn't block a merge.** To require code-owner approval, enable that requirement in branch protection or a ruleset for the target branch. Check bypass permissions as well.

## Two details worth checking

One detail is easy to miss: **the last matching pattern wins.** For a change only inside `/infra/`, GitHub requests the platform team, not the application team as well. The default rule doesn't get added to the more specific one.

If both teams should be listed for infrastructure, put them on the same line:

```text
/infra/                 @example-org/application @example-org/platform
```

Even then, the standard code-owner review requirement doesn't require approval from both teams. Either listed owner can satisfy it for those files. If your policy says both teams must sign off, this line alone won't enforce that.

That's worth testing before telling the teams their approval policy is enforced.

## Review ownership isn't service ownership

For `invoice-orbit`, there are three different questions:

| Question | Where the answer belongs |
| --- | --- |
| Who can read or change the code? | Repository permissions |
| Who maintains the service and handles incidents? | A team agreement recorded in the README or service catalog |
| Who gets asked to review these files? | `CODEOWNERS` |

The platform team can review `/infra/` without taking responsibility for dependency updates or production incidents across the application. Adding a team to the file shouldn't be the first time it hears about the work.

## Does every repository need one?

If several teams contribute to a repository, or people regularly ask who should review a change, I'd add one. It's also useful when a particular area needs specialist knowledge, such as deployment configuration or authentication code.

A new contributor benefits even if everyone else already knows who to ask. They can open a PR without first finding someone who knows the team structure.

But **no, I wouldn't add it to every private repository by default**. If one small team already routes reviews reliably and doesn't need enforced owner approval, the file is optional. A default owner can still be convenient; directory-by-directory rules probably aren't.

Keep the specialist rules narrow. Naming platform on every file pulls that team into unrelated changes and can create a review queue without improving the review.

Use teams rather than individuals when the responsibility belongs to a team. Keep memberships and patterns current when teams rename or services move; stale rules won't reliably reach the people you need.

## Rolling it out without creating a review backlog

I'd try this with a few actively maintained repositories before generating files across the organization.

1. Agree who maintains the service and which changes need specialist review.
2. Add the smallest useful set of patterns, including ownership of `CODEOWNERS` itself.
3. Check team access and any errors GitHub highlights in the file. Once it's on the base branch, open representative PRs and confirm the review requests.
4. If approval must be mandatory, configure the branch rule and verify bypass permissions.
5. After teams have used it, check whether requests reach the right people and whether reviews are waiting longer. Revisit the rules when responsibilities change.

Back at the opening PR, the author marks the `/infra/` change ready for review and platform gets the request. Nobody has to ask in chat. Application still maintains the billing service; we've made one part of contributing to it less dependent on knowing who to call.

## References

- [GitHub: About code owners](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners)
- [GitHub: About protected branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)
- [GitHub: About rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets)
