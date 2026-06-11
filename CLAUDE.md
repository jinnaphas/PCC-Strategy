# CLAUDE.md — PCC Strategy Site

Project memory for Claude Code. Read this first on every session.

- **Live site:** https://jinnaphas.github.io/PCC-Strategy/ (GitHub Pages, served from `main`)
- **Repo:** jinnaphas/PCC-Strategy
- **Owner communicates in Thai.** Reply in Thai unless asked otherwise.

---

## 0. Workflow Rules (NON-NEGOTIABLE)

The site is a **single 7 MB `index.html`** edited from **two** channels that both
write to `main`:
1. **Edit via Browser** — in-page ✏️/💾 system that PUTs to the GitHub API (`branch:'main'`).
2. **Claude Code** — this environment, plain `git`.

Because both channels land on `main`, follow these rules to avoid clobbering
browser edits (a merge conflict on a 7 MB single file is brutal):

1. **Always `git pull origin main` BEFORE editing.** Browser edits may have landed since.
2. **Commit + push straight to `main`.** The owner wants live updates immediately. (Authorized 2026-06-11.)
3. **NEVER force-push** unless explicitly told. A blind `--force` wipes browser edits.
4. **Never delete `assets/` or `.nojekyll`.** 74 images live in `assets/` (img001–img074).
5. Keep `index.html` valid: `#hash-clear` first in `<head>`, `#pcc-auth` first in `<body>`,
   `#gh-edit-js` last in `<body>`, file ends with `</body></html>`.

Standard loop: `git pull origin main` → edit → `git add -A && git commit` → `git push origin main`

---

## 1. Repo Structure

```
PCC-Strategy/
├── index.html      # ~7 MB single-page app — THE source of truth
├── assets/         # 74 images (img001–img074, .jpg/.png) — never drop these
├── .nojekyll       # disables Jekyll on GitHub Pages
└── CLAUDE.md       # this file
```

Editing such a large file: don't read the whole thing into context. Use `grep`/`Grep`
to locate, then targeted offset reads or Python slice-and-replace for surgical edits.

---

## 2. Architecture — 3 Injected Scripts

| Script id | Location | Purpose |
|-----------|----------|---------|
| `#hash-clear` | first in `<head>` | Clears URL hash before render so modals don't auto-open on refresh |
| `#pcc-auth` | first in `<body>` | Login overlay (fullscreen), SHA-256 password check, session in localStorage |
| `#gh-edit-js` | last in `<body>` | In-browser edit mode + save-to-GitHub via API |

### Login (`#pcc-auth`)
- Users JPP and PCC. Passwords are stored only as **SHA-256 hashes** in the script
  (plaintext passwords are NOT recorded here — keep it that way).
- Session: localStorage `pcc_session` = `{user, t}`, 4-hour timeout (`SESSION_HOURS = 4`).
- Add a user: compute `sha256(password)`, add to the `USERS` object, push. (Requires redeploy.)

### Edit + Save (`#gh-edit-js`)
- Config: `var GH={owner:'jinnaphas',repo:'PCC-Strategy',branch:'main',api:'https://api.github.com',file:'index.html'}`
- ✏️ toggles `contentEditable` on whitelisted selectors; 💾 runs `doSave()`:
  clones DOM → strips injected UI (`#gh-fab`,`#gh-cfg-btn`,`#gh-status`,`#gh-panel`),
  `style[data-gh-edit]`, `contenteditable` attrs, and **`open` on every `.modal-overlay`**
  → base64 → PUT to GitHub.
- ⚙️ gear = token panel; PAT saved to localStorage `pcc_gh_tok`.
- Editable selectors: `.p2-name/.p2-desc/.p3-*/.p4-*/.p1-title/.p1-sub/.sl-card-name(-th)/.sl-desc-en(-th)/.obj-headline/.modal-title`, modal text nodes, `.nav-strategy-label/.nav-strategy-sub`.

---

## 3. Known Bug Fixes — Do Not Reintroduce

1. Modal auto-opened on refresh → removed baked-in `open` class + added `#hash-clear` first in head.
2. "Encoding HTML…" stuck on screen → `doSave()` clones & strips UI **before** encoding.
3. Edit FAB missing → `#gh-edit-js` had been corrupted (no closing `</script>`); rebuilt clean.
4. Modal re-opened after save → `doSave()` strips `open` from all `.modal-overlay`.
5. `assets/` deleted → caused by force-push from a partial clone; never force-push, never partial-clone here.

Invariant to keep green: `grep -c 'modal-overlay open' index.html` must be **0**.

---

## 4. Security Notes

- **No secrets in the repo.** Never commit the PAT or plaintext passwords into any tracked file.
- The PAT lives only in the browser's localStorage (`pcc_gh_tok`), entered via the ⚙️ panel.
- If a PAT is ever exposed, rotate it at GitHub → Settings → Developer settings, then re-enter via ⚙️.

---

## 5. Known Cleanup / Pending Work

- [ ] Strip stray `<chatgpt-sidebar ...></chatgpt-sidebar>` artifact near `</html>` (browser-extension junk baked in during a save).
- [ ] Login logging (options brainstormed: Google Sheets/Apps Script, GA4, webhook, Notion).
- [ ] Password/user management without full redeploy.
- [ ] Show logged-in user + remaining session time on page.
- [ ] Expand editable selectors as content grows.
