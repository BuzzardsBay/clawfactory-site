# Close-out: make the download buttons download the installer directly

**Date:** 2026-08-29
**Repository:** `BuzzardsBay/clawfactory-site` (the repo that serves `clawfactory.app`)
**Commits:** `142ec82` (the change), plus the commit that adds this file
**Status:** COMPLETE. Change is live and verified against the live site.
**Product repo touched:** NO. `ClawFactory-Secure-Setup` was read only.

---

## 1. Why the job existed

All three download links on the live site pointed at
`https://github.com/BuzzardsBay/clawfactory-secure-setup/releases/latest`. That lands a
visitor on a GitHub release page where the installer sits below the fold under a collapsed
Assets list. A non-technical visitor does not know to scroll, and the page reads like a
developer artefact rather than a download.

## 2. Establishing the real state, by measurement

The governing risk here was the citation clause added in `fa4423f`: a previous session
asserted the state of this site from a file in the wrong repository. So the first act was to
prove which repository serves the domain, and then to prove that the file being edited is
the file that is served.

### 2.1 Which repository serves the domain (GitHub Pages API)

```
$ gh api repos/BuzzardsBay/clawfactory-site/pages
{"status":"built","cname":"clawfactory.app","html_url":"https://clawfactory.app/",
 "build_type":"legacy","source":{"branch":"main","path":"/"},"public":true,
 "https_enforced":true}

$ gh api repos/BuzzardsBay/clawfactory-secure-setup/pages
{"message":"Not Found","status":"404"}
```

`clawfactory-site`, branch `main`, path `/`. The product repo has no Pages site at all.

### 2.2 The stronger check: served bytes equal repo bytes

```
sha256 https://clawfactory.app/          b28b2aca72233cae0dbf583d9610259a8ac32796ba926c9973f5a7f3937c0add
sha256 clawfactory-site/index.html       b28b2aca72233cae0dbf583d9610259a8ac32796ba926c9973f5a7f3937c0add
```

Identical. There is no build step and no transformation between repo and live page, so
"the file I am editing" and "the file that produces the live page" are the same object, not
two objects that happen to look alike.

### 2.3 Link census: live page against repo source

Three release-page links on the live page, three in the source, at the same line numbers,
in a file whose hash matches the served bytes. Count matched, so the file was the right one.

| # | Link text on the live page | Where |
|---|---|---|
| 1 | `Download` | fixed nav bar, top right |
| 2 | `Download for Windows` | hero primary CTA |
| 3 | `Get ClawFactory-Secure-Setup.exe from the releases page` | How It Works, Step 01 |

### 2.4 The asset filename, verified rather than assumed

```
$ gh api repos/BuzzardsBay/clawfactory-secure-setup/releases/latest
tag: v1.4.4   published: 2026-08-29T15:36:10Z
assets:
  ClawFactory-Secure-Setup.exe    440610608    application/x-msdownload
```

One asset, named `ClawFactory-Secure-Setup.exe`. The job brief guessed this name correctly,
but the guess was checked, because the direct-download URL matches an asset by literal
filename and a wrong name produces a 404 on the most important link on the site.

## 3. What changed

One file: `index.html`. Thirty-two lines added, three removed.

### 3.1 The three primary links

Old, all three identical:

```
https://github.com/BuzzardsBay/clawfactory-secure-setup/releases/latest
```

New, all three identical:

```
https://github.com/BuzzardsBay/clawfactory-secure-setup/releases/latest/download/ClawFactory-Secure-Setup.exe
```

### 3.2 One anchor-text correction

Link 3 read `Get ClawFactory-Secure-Setup.exe from the releases page`. The phrase described
where the link went and the link no longer goes there, so leaving it would have made the
sentence false. It now reads `Get ClawFactory-Secure-Setup.exe`. This is the only copy
change beyond the link targets, and it is inside the link.

### 3.3 Three secondary links

The site tells visitors to check the SHA-256 of the installer, and that value lives on the
release page. Sending everyone straight at the file would have removed the only route to
it. So each download button gained a footnote link, identical wording in all three places:

```html
<a class="secondary-link" href="https://github.com/BuzzardsBay/clawfactory-secure-setup/releases/latest">Release notes and SHA-256</a>
```

Type scale, measured from the rendered DOM rather than read off the stylesheet:

| Place | Primary | Secondary |
|---|---|---|
| Nav | 13px | 12px |
| Hero | 16px | 12.5px |
| Step 01 | 15px | 12.5px |

12.5px is the existing `.cta-tier-note` size, so the footnote sits on the page's own scale.
Underlined, `--text-dim`, no border, no button styling. A footnote, not a second button.

### 3.4 One layout deviation, declared

The nav is a single-line fixed bar using `justify-content: space-between`. "Under" is not
physically available there, so the nav footnote sits beside the pill, to its left. Below
768px there is no room for it alongside the wordmark and the pill, so it is hidden at that
breakpoint. The hero footnote is a few hundred pixels below it on the same screen, so the
release page stays one tap away on mobile.

Verified at three widths, from the rendered DOM:

- 1290px: wordmark 97 to 161, footnote 866 to 1046, pill 1060 to 1153. No collision.
- 820px, just above the breakpoint: wordmark 24 to 161, footnote 493 to 673, pill 687 to 781. No collision.
- 375px: nav footnote hidden, hero and step footnotes visible, `scrollWidth == clientWidth`.

No horizontal overflow at any of the three.

### 3.5 The hero copy line: checked, unchanged

> Version 1.4.4 (middot) about 440 MB, because the Linux filesystem and the management app
> are bundled rather than fetched. The release page lists the SHA-256 of the installer so
> you can check the file you downloaded is the file we built.

Version: present. Size: present, and 440 MB is accurate against the measured 440,610,608
bytes. Windows: the button directly above it says "Download for Windows" and the line
directly below says "Windows 10/11". A visitor can tell before clicking what they are
getting. Nothing was added.

## 4. Verification

### 4.1 The direct URL, before committing

```
HTTP/1.1 302 Found
Location: .../releases/download/v1.4.4/ClawFactory-Secure-Setup.exe

HTTP/1.1 302 Found
Location: https://release-assets.githubusercontent.com/github-production-release-asset/1222707440/...

HTTP/1.1 200 OK
Content-Length: 440610608
Content-Disposition: attachment; filename=ClawFactory-Secure-Setup.exe
Content-Type: application/octet-stream
```

`Content-Length` equals the asset size reported by the API, to the byte, so the URL resolves
to the real installer and not to an error page. `Content-Disposition: attachment` means the
browser downloads rather than navigates. One byte was transferred, using a `Range: 0-0`
request.

### 4.2 Rendered output, not source

There is no build step, so "the built output" is the served file. To read rendered targets
rather than source text, the edited file was served over local HTTP and the anchors were
read out of the live DOM. Six links, all visible: three direct to the installer, three to
the release page. Reported in full in section 5.

### 4.3 The dash check, and a note on how it nearly went wrong

The brief forbids em dashes. The first check used `grep -P` and failed with
`-P supports only unibyte and UTF-8 locales`, at which point its `|| echo NONE` fallback
printed a clean result. That reading was a measurement failure wearing the costume of a
pass, and it was discarded.

The replacement checked bytes directly over the 32 added lines: 0 occurrences of U+2014,
0 of U+2013, 0 of the HTML entities for either. Per the audit-regex-is-itself-a-probe clause
in the preamble, both patterns were first fired against a planted canary and both returned
1, so the zero is a measurement and not a silent grep failure.

### 4.4 Line endings

This repository has a history of CRLF corruption that `git status`, `git diff` and `grep`
are all blind to. Checked explicitly: 0 CR bytes in the file, and `git ls-files --eol`
reports `i/lf w/lf`. The trap did not fire.

## 5. Post-deploy readings from the live site

Pushed `142ec82`. Pages reported `built` at that commit, but the build status is not the
same claim as the CDN serving it, and this site has served stale for 32 consecutive polls
before. So the live URL was polled until the served bytes matched.

```
MATCH on poll 1   sha=214db95ad1741fbd8d98d181f96d2863e5256ab00fdfa1e94ddfac6ee6168a76
```

The deploy landed on the first poll. Live bytes were then re-fetched anonymously and the six
links extracted from those bytes, not from the local file:

**Three primary links, HEAD following redirects:**

```
reading 2:  HTTP/1.1 200 OK
            Content-Length: 440610608
            Content-Disposition: attachment; filename=ClawFactory-Secure-Setup.exe
reading 3:  HTTP/1.1 200 OK
            Content-Length: 440610608
            Content-Disposition: attachment; filename=ClawFactory-Secure-Setup.exe
reading 5:  HTTP/1.1 200 OK
            Content-Length: 440610608
            Content-Disposition: attachment; filename=ClawFactory-Secure-Setup.exe
```

Each is a 302 to `/releases/download/v1.4.4/ClawFactory-Secure-Setup.exe`, then a 302 to
`release-assets.githubusercontent.com`, then 200 with `Content-Type: application/octet-stream`.

**Three secondary links, GET following redirects:**

```
reading 1:  HTTP 200  redirects=1  final=https://github.com/BuzzardsBay/clawfactory-secure-setup/releases/tag/v1.4.4
reading 4:  HTTP 200  redirects=1  final=https://github.com/BuzzardsBay/clawfactory-secure-setup/releases/tag/v1.4.4
reading 6:  HTTP 200  redirects=1  final=https://github.com/BuzzardsBay/clawfactory-secure-setup/releases/tag/v1.4.4
```

All six as specified.

## 6. STANDING OBLIGATION: this URL is filename-coupled

`https://github.com/BuzzardsBay/clawfactory-secure-setup/releases/latest/download/<name>`
resolves `latest` dynamically, but it matches the asset by **literal filename**.

**It survives a future release only while the installer keeps the exact name
`ClawFactory-Secure-Setup.exe`.**

If a future release ships `ClawFactory-Secure-Setup-v1.5.0.exe`, or renames the asset for
any reason, all three primary links on `clawfactory.app` return 404 and the site's most
important button breaks. Nothing in the release pipeline checks this, and the failure is
silent: the site keeps serving, the button keeps looking like a button, and the click
produces a GitHub 404.

Two ways to discharge this, either is sufficient:

1. Keep the asset name constant across every release. It has been constant so far.
2. Add a post-release check that fetches the direct URL and asserts a 200 with a
   `Content-Length` matching the release asset, and treat a 404 as a release blocker.

Until one of those exists, this is a manual obligation on whoever cuts the next release.

## 7. Scope discipline

- **Product repo:** read only. `docs/VALIDATION_PREAMBLE.md` was read. Nothing was written.
- **No build, no release, no tag.** The `v1.4.4` release was read through the API only.
- **Site copy:** unchanged except the link targets and the one anchor-text correction in 3.2.
- **Publishing:** the session stopped before the push and printed a full operator card. The
  push happened only on explicit approval.

## 8. Where this file lives, and why

The brief asked for `docs/session_reports/` in this repo if that convention exists here. It
does not: before this commit the repository tracked exactly four files, `index.html`,
`CNAME`, `README.md` and `.gitignore`, with no directories at all.

The directory was created here anyway, rather than in the product repo, because this job's
brief explicitly forbids changes to the product repo.

**Consequence worth knowing:** this repository is public and is the Pages source, so this
file is fetchable under `clawfactory.app/docs/session_reports/`. Everything in it is already
public: public repo names, public commit hashes, public URLs, and the size of a public
installer. There are no credentials, hostnames, internal paths or infrastructure details in
it. It carries no YAML front matter, so Jekyll copies it verbatim without Liquid processing,
which is the lowest-risk way to add a file to a working Pages site.

If session reports should not be served from the marketing domain, the fix is a one-line
`_config.yml` with an `exclude:` entry, or moving these to the product repo. Both are out of
scope for this job.

## 9. Commits

Two, and the reason there are two rather than one:

1. `142ec82`, the site change, one commit as the brief specified.
2. This close-out, which could not be in that commit because it reports readings taken from
   the live site after that commit was pushed and deployed. A close-out committed before the
   deploy would have had to claim a result it had not yet measured.

Explicit per-file staging both times. No `git add -A`. No tag.
