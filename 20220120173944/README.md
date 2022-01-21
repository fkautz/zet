# Zet as a Service

We can produce an environment that allows a simple Zet as a Service to be created with GitHub's in-browser file creation, search, GitHub Actions, and GitHub Pages.
This approach may be replicated in other git environments such as gitlab or your only fully hosted environment.
Ideally, the tooling should be github agnostic.

A simple webpage acts as a landing zone.
Anyone may download and host their own zet page.
The final site is deployable to any static host, e.g. github pages, netlify, Cloudflare Pages.

The webpage has four functions:

- Link to create a new zet. e.g. https://github.com/fkautz/zt.dev/new/main (see bug beloow)
- Search box that links to GitHub search
- Jump to most recent zet (github actions can help here)
- List most recent zets (pull via rss feed)

Pulling in RSS feeds from other sites may be possible.
We could register zets for others to read.

## GitHub Bug:
GitHub disregards the last directory if you provide a filename.
	https://github.com/fkautz/zt.dev/new/main/01234567890123?filename=foo.md results in zt.dev/foo.md
	https://github.com/fkautz/zt.dev/new/main/01234567890123/?filename=foo.md results in zt.dev/foo.md
	https://github.com/fkautz/zt.dev/new/main/01234567890123/discarded?filename=foo.md results in zt.dev/01234567890123/foo.md
