# Security policy

## Scope

Security fixes apply to the current site configuration and build/deployment
workflow on the default branch. Older revisions are not maintained separately.

## Report a vulnerability

Please use GitHub's private vulnerability reporting for this repository:

[Report a vulnerability privately](https://github.com/LuanErasmus/luanerasmus.github.io/security/advisories/new).

Include the affected page or file, reproduction steps, and the likely impact.
Do not include live credentials, personal data, or information belonging to
someone else. Do not open a public issue containing vulnerability details or
secrets.

If the issue is in Hugo, Blowfish, or another upstream dependency, use that
project's security reporting process as well.

## Handling exposed credentials

Revoke or rotate an exposed credential immediately. Removing it from the latest
commit does not remove it from Git history, logs, artifacts, or existing copies.
Report the location privately without copying the credential into the report.

## Repository protections

Security scans and an empty alert list do not guarantee that a repository is
free of secrets or vulnerabilities.
