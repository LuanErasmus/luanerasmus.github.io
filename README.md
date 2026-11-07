# Luan Erasmus

A simple personal blog built with [Hugo](https://gohugo.io/) and
[Blowfish](https://github.com/nunocoracao/blowfish) v3.9.0, published with GitHub
Pages. The GitHub-inspired dark palette keeps the focus on Blogs, Projects, Events,
AboutMe, and Github. The homepage combines a compact profile with **What's New**
cards for the latest blog post and project.

The theme is pinned in `go.mod`, with dependency checksums in `go.sum`. No npm
installation or frontend build tool is required.

The shared header logo also supplies the SVG browser-tab icon. AboutMe uses a
centered reading column with left-aligned text. Articles use a centered reading
area up to 56rem wide, with fluid page gutters that leave more room on tablets
and smaller screens. Blog and project listings use the theme's responsive card
grid, and the homepage reuses its Tailwind grid and flex utilities. Blowfish
already includes Tailwind, so Bootstrap is not loaded alongside it; custom CSS
is reserved for site-specific appearance, article sizing, and workflow diagrams.

## Run locally

Install Hugo Extended 0.167.0 and Go 1.27.3, then run:

```sh
go mod download
hugo server
```

Open the local address printed by Hugo. Add blog posts under `content/blog/` and projects under `content/projects/` as Markdown files with front matter.

The Events page is linked from the main menu. A Next Event card on the homepage
links to that page and selects the nearest event dated today or later.
Add entries to the `events` list in `content/events.md`, with a `title`, a
`date` in `YYYY-MM-DD` format, and a `role` such as `Speaking`. Entries are shown
in date order. Event dates are separate from Hugo publication dates, so upcoming
events remain visible in production builds. Selection uses the build's current
date; rebuild the site after an event to refresh the homepage card. If there are
no upcoming events, the homepage displays a short message and the Events link.
An event can also include a `session` title and `summary` for both cards.
Optional `overview`, `audience`, `takeaways` (title/description pairs), `demo`,
and `resources` (title/URL pairs) appear only on the Events page, keeping the
homepage preview compact.

Projects use thumbnail cards. Keep each project in a page bundle containing
`index.md` and its GitHub social-preview image as `feature.png` or `feature.jpg`. Set
`featureimage` to that filename in the front matter. The `gh-action-inspector` image
is a local copy of the repository's configured GitHub social preview, so builds
and visitors do not depend on GitHub's image server. Replace it when that preview
changes.

The `agentic-blueprints` project also uses a local copy of its repository's
social-preview image, stored as `feature.jpg` in its page bundle.

Homepage cards automatically select the newest published entry in each section
by its front-matter date. Drafts and future-dated entries are excluded from
production builds. Entries with an empty body show **Coming soon**; the badge
disappears when content is added.

To create a draft post:

```sh
hugo new content blog/my-first-post.md
```

Preview drafts with `hugo server --buildDrafts`. Set `draft: false` when a post is
ready to publish.

Use `hugo --gc --minify` to produce the production site in `public/`.

## Theme updates

Dependabot opens weekly update pull requests for the theme and GitHub Actions.
All dependency updates are grouped per ecosystem: GitHub Actions and Go modules.
Security updates are also grouped per ecosystem, separately from regular
version updates as required by Dependabot.
Review the theme's supported Hugo versions before merging an update; update the
Hugo version in the workflow and these instructions together when necessary.
Hugo, actionlint, and Gitleaks binaries are downloaded over HTTPS and checked
against reviewed SHA-256 digests before extraction. When updating any tool,
update its pinned version and Linux archive digest together. Dependabot does not
update these binary downloads; review their upstream releases manually. Never
obtain the expected digest from the same untrusted download during a build.

## Publish

GitHub Actions handles both building and deployment. Pull requests to `main` run
the **Build Hugo site** check without deployment permissions. Merging to `main`
builds the site and deploys the generated `public/` artifact to GitHub Pages.
Manual deployment is also available through `workflow_dispatch` on `main`.

In the repository settings, set **Pages → Build and deployment → Source** to
**GitHub Actions**. Keep the `github-pages` environment restricted to `main`.

Changes should go through a pull request, not a direct push to `main`. All Actions
are pinned to full commit SHAs, including nested Actions used by the Pages
artifact uploader. Check those nested references when reviewing Action updates:
pinning a composite Action does not pin its dependencies. Checkout does not
retain Git credentials, and Pages/OIDC write permissions are limited to the
deployment job.
The required build also lints the workflow, checks dependency integrity, and
scans reachable Git history for secrets with redacted Gitleaks output.

## Security

Use [the security policy](SECURITY.md) for private vulnerability reports.
