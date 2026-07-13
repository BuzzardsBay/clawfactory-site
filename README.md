# clawfactory-site

Public marketing site for **clawfactory.app**, served by GitHub Pages from this
repo (branch `main`, root folder).

## Standing rule: marketing sites and product source never share a repo

**Never add installer/product source to this repo. It is marketing-site-only,
public, forever.**

This repo exists ONLY to host the public website. The ClawFactory product /
installer source lives in a separate PRIVATE repo and must stay separate.

Why this rule exists: the site previously lived inside the installer repo's
`docs/` folder. On 2026-05-22 that repo was switched to private, and because
GitHub Free does not serve Pages from a private repo, the site silently went
down and stayed down for about seven weeks (2026-05-22 .. 2026-07-10). That
outage is card #82: making `clawfactory-secure-setup` private silently killed
clawfactory.app for seven weeks. Hosting the public site in its own public repo
means privatizing the product source can never take the website down again.

## Contents

- `index.html` -- the landing page, copied as-is from the installer repo
  (last edited 2026-05-14). Refreshing the copy is separate launch-package work.
- `CNAME` -- custom domain (clawfactory.app).

## DNS

Apex A records point at the GitHub Pages IPs; `www` CNAME points at
buzzardsbay.github.io. DNS was never the problem and is unchanged.
