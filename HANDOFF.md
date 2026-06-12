# PCC Strategy Site — Handoff

**Updated:** 2026-06-12 · **Live:** https://jinnaphas.github.io/PCC-Strategy/ · **Repo:** jinnaphas/PCC-Strategy (public, default `main`)

This is the onboarding handoff for anyone (human or a new Claude Code session) picking up the
project. **Read `CLAUDE.md` first** for the non-negotiable workflow rules; this file adds the
fuller narrative: current state, what's been done, and the roadmap. Owner communicates in Thai.

---

## 1. What this is
A single ~6.9 MB static `index.html` (a 4-phase corporate-strategy microsite) served by GitHub
Pages from `main`. It is edited from **two channels that both write to `main`**:
1. **Edit via Browser** — an in-page ✏️/💾 system that PUTs `index.html` to the GitHub API.
2. **Claude Code** — plain `git`.

```
PCC-Strategy/
├── index.html   # THE source of truth (~6.9 MB, 16 <script> blocks)
├── assets/      # 74 images img001–img074 — never delete
├── .nojekyll    # disables Jekyll on Pages — never delete
├── CLAUDE.md    # always-read project memory (rules, architecture)
└── HANDOFF.md   # this file
```

## 2. Working rules (summary — full version in CLAUDE.md)
1. **`git pull origin main` BEFORE editing** (browser edits may have landed).
2. **Commit + push straight to `main`** (owner wants live updates; authorized 2026-06-11).
3. **Never force-push, never partial-clone** (both have wiped `assets/` before).
4. Don't read the 7 MB file whole — `grep`/anchors → targeted reads or Python slice/regex with
   assertions. Keep script order valid (`#hash-clear` head-first, `#pcc-auth` body-first,
   `#gh-edit-js` last script; file ends `</body></html>`).
5. **Health invariants (keep green):** 0 duplicate element IDs · `grep -c 'modal-overlay open'`
   = 0 · `<script>`/`</script>` balanced · file ends `</body></html>`.

Environment note: this runs in **Claude Code** (repo cloned fresh each session, full `git` +
file tools; GitHub ops via `mcp__github__*`). The old Cowork `/tmp` partial-clone workflow is
obsolete — do not use it.

## 3. Architecture — 3 injected scripts
| id | location | purpose |
|----|----------|---------|
| `#hash-clear` | first in `<head>` | clears URL hash before render → no modal auto-open on refresh |
| `#pcc-auth` | first in `<body>` | login overlay, SHA-256 check, session in localStorage `pcc_session` (4 h) |
| `#gh-edit-js` | last script in `<body>` | in-browser edit mode + save-to-GitHub |

The other ~13 inline scripts are page logic (modals, nav, changelog renderer).

**`#gh-edit-js` (hardened — see §4):** config `GH={owner,repo,branch:'main',api,file:'index.html'}`.
✏️ enter edit (contentEditable on the `SELS` whitelist) / 💾 save+exit / ✖ `#gh-cancel` discard
without saving (restores `el.__ghOrig` snapshot). `doSave()` clones the DOM, strips the edit UI
(`#gh-fab,#gh-cfg-btn,#gh-status,#gh-panel,#gh-cancel`) **plus runtime-injected nodes
(`#cl-modal-new`, `#pcc-login-overlay`)**, `style[data-gh-edit]`, `contenteditable` attrs, and
`open` from every `.modal-overlay`, then base64 → PUT. **Dirty-check:** skips the commit unless
an `input` event fired during edit. PAT is entered via the ⚙️ panel, stored in localStorage
`pcc_gh_tok` only. ⚠️ **If you add JS that injects DOM at runtime, add its id to the strip list
in `doSave`** or a browser-save bakes duplicates into the static HTML.

**Change Log (`CL_DATA`):** array delimited by `/*==CL_DATA_START==*/ … /*==CL_DATA_END==*/`.
Schema `{id,date,cat,detail,area,by,status,snap}`, `cat ∈ {strategic,structural,personnel,data,
content,system}`, date `'DD <เดือนย่อ> YY'`. `openCLModal()` rebuilds the modal fresh at runtime
(drops any stale copy); `saveCLToHTML()` regenerates the block — **keep the markers intact** when
hand-editing. Latest id = 17.

## 4. Work done so far
| commit | summary |
|--------|---------|
| `0aef8a7` | add `CLAUDE.md` (project memory + main-deploy workflow) |
| `ae085c4` | **cleanup (Set A):** removed duplicate modal-quality injector + dead changelog (`dlg-cl`, static `cl-modal-new`), fixed 22 duplicate IDs, stripped `<chatgpt-sidebar>` artifact (~139 KB smaller) |
| `613802c` | **save hardening (Set B):** strip injected nodes before save, dirty-check (no empty commits), cancel button; Change Log entries 15–17 |
| `9f19bfb` | document hardened `doSave` in `CLAUDE.md` |

**Bug history — do not reintroduce:** modal auto-open (fixed via `#hash-clear` + no baked `open`);
"Encoding…" stuck (clone/strip before encode); missing FAB (corrupted `#gh-edit-js`, rebuilt);
modal reopen after save (strip `open`); `assets/` deleted (force-push/partial-clone); file bloat
each browser-save (runtime-injected nodes baked in — fixed in Set A/B).

## 5. Security — current limitation & roadmap (DECISION PENDING)
Because the site is a public static page, all content ships to the browser; the `#pcc-auth`
overlay gates the view but not the underlying source. This is a known limitation. Owner wants to
**stay on GitHub Pages** and is still choosing among:
- **Option 1 — soft gate + login logging.** Strengthen passwords; POST `{user,time}` on login to
  a free endpoint (Google Apps Script→Sheet, or a Make/webhook). Low effort, no Edit-flow impact,
  delivers the long-wanted login log. Content remains public. *(Recommended first step.)*
- **Option 2 — client-side encryption (StatiCrypt-style).** Ship encrypted HTML, decrypt in-browser
  with a passphrase → source becomes unreadable. Trade-offs: single shared passphrase, no per-user
  log, the Edit/Save flow must re-encrypt before push, and lose-the-passphrase = lockout (keep an
  offline plaintext backup).
- **Option 3 — Cloudflare Pages + Access (real auth).** Best protection, free ≤50 users, manage
  users without redeploy, git workflow unchanged. Requires moving off GitHub Pages (URL changes) →
  on hold per owner's preference.

## 6. Secrets handling (IMPORTANT)
- **Never commit a PAT or plaintext password** to this (public) repo. The PAT lives only in the
  browser's localStorage `pcc_gh_tok`, entered via the ⚙️ panel.
- A write-scoped PAT was shared in plaintext in the original handoff — **rotate it** (GitHub →
  Settings → Developer settings) and pass the new token, plus current login credentials, to the
  next operator **out-of-band** (not in any file in this repo).

## 7. Pending / next
- [ ] Pick & implement Set C (likely start: Option 1 logging — MCP `Google_Drive`/Sheets or `Make` are available).
- [ ] User/password management without redeploy (Option 3 solves natively).
- [ ] Show logged-in user + remaining session time on page.
- [ ] Expand `SELS` editable selectors as content grows.

## 8. Handy commands
```bash
git pull origin main
grep -oE 'id="[^"]+"' index.html | sort | uniq -d        # must be empty
grep -c 'modal-overlay open' index.html                  # must be 0
python3 -c "import hashlib;print(hashlib.sha256('PWD'.encode()).hexdigest())"   # new user hash
git add -A && git commit -m "..." && git push origin main
```
Useful skills: `/security-review` (before auth/crypto changes), `/verify` (browser smoke-test),
`/code-review`. GitHub ops via `mcp__github__*`.
