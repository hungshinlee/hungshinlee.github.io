# CLAUDE.md

Working notes for this repo. Everything lives in `index.html`: markup, inline styles, and
the content data. There is no build step, no package manager, and no test suite.

## Preview

```sh
python3 -m http.server 8899
```

`file://` does not work — `support.js` needs an HTTP origin.

## Deploying is two steps, not one

Pushing to `main` publishes only the mirror. The canonical URL is
`https://web.ntnu.edu.tw/~hslee/`, and NTNU has no CI hook, so the real site does not
change until `index.html` is uploaded by hand:

```sh
curl -T index.html --ssl-reqd -k -u hslee ftp://web.ntnu.edu.tw/<webroot>/
```

The host is Pure-FTPd on port 21 with SSH closed, so `scp`/`sftp` are unavailable.
Plaintext FTP is refused (`421 ... reconnect using TLS`) and the certificate is
self-signed, hence `--ssl-reqd -k`. **The owner runs this personally — never handle the
FTP password.** Passing `-u hslee` without a password makes curl prompt interactively,
which also keeps it out of shell history.

Confirm an upload landed by comparing hashes; the page is one self-contained file, so
they should match exactly:

```sh
shasum -a 256 index.html
curl -s https://web.ntnu.edu.tw/~hslee/index.html | shasum -a 256
```

Both hosts serve the identical file, so `canonical`, `og:url`, and `og:image` point at
NTNU even when the page is fetched from github.io. That is deliberate — it consolidates
search ranking on the canonical host.

## Where content lives

Near the bottom of `index.html`: two language dictionaries, `ZH` and `EN`, plus data
arrays — `PILLARS`, `CAREER`, `PUBS`, `PROJECTS`, `COURSES`, `AWARDS`, `HIGHLIGHTS`,
`NEWS`. Every array entry carries both languages, and the render function picks one.
Adding content means editing an array, not the markup.

`PUBS` is rendered in array order — nothing sorts it, so new entries go where they
should appear. Its conventions, which the byline rendering depends on:

- Titles in **sentence case** (`Anchoring speech with semantics: A multimodal…`), matching
  the existing entries.
- Authors as initials: `K.-T. Huang, C.-Y. Yang, H.-S. Lee, B. Chen`. The exact string
  `H.-S. Lee` is what the data layer searches for to bold the site owner's name — a
  different spelling silently loses the emphasis.
- `venue` is a short tag (`LREC`, `IEEE ICASSP`). Qualify it only when the distinction
  carries weight, as with `EMNLP (Main Conference)` versus Findings.
- `corr: true` adds a 通訊作者 badge. `url` is optional.

## Runtime gotchas

These each cost real debugging time, so they are worth knowing before editing.

**`<title>` and `viewport` belong in the static `<head>`, not `<helmet>`.** The runtime
injects `<helmet>` children into `document.head` after JS runs, so crawlers — LINE,
Facebook, Slack, Google — never see anything declared there. That is why the Open Graph
tags are static. Keep exactly one `<title>`; the first in tree order wins.

**`{{ }}` escapes HTML.** To emphasise part of a sentence, split the string into segments
and wrap one in markup, the way the hero headline (`heroTitleA` / `heroTitleKey` /
`heroTitleB`) and `HIGHLIGHTS` (`pre` / `venue` / `mid` / `track` / `post`) both do. Keep
the interpolations and tags on one line — a line break between them renders as a space.

**`opacity` on a parent dims its children**, so a nested `<strong>` cannot be brighter
than its container. Use `color: rgba(…, .66)` on the parent instead when a child needs
its own colour.

**`sc-for` renders a React Fragment**, producing no DOM node of its own. Loop items are
therefore direct children of the wrapper, and `display:flex; gap:…` on that wrapper
behaves as expected.

**A missing attribute value is omitted, not emptied.** A `PUBS` entry without `url`
renders its title as plain text rather than a link pointing at the current page.

## Responsive approach

Fluid rather than breakpoint-driven: `clamp()` for type and spacing, `flex-wrap` with
flex-basis thresholds, and `grid auto-fit` with `minmax(min(100%, Npx), 1fr)`. There are
no layout media queries and adding one should be a deliberate choice — the only two
present are `prefers-reduced-motion` and `print`. `min-width: 0` on flex children is
load-bearing; removing it lets long text overflow.

Verified clean down to 320px. When changing layout, check that width rather than trusting
the desktop view.
