# TermDecks working notes

Single-file web app (`index.html`, vanilla JS + CSS + HTML, no build, no deps). It builds printable, poker-sized (2.5″ × 3.5″) terminal-command reference cards, with an on-screen preview, a 3×3 print sheet, and a Leitner spaced-repetition quiz. State lives in `localStorage` key `cardcheat_v3`.

## Writing style: never use em dashes

No em dashes (`—`) anywhere a person reads: UI strings, card text, README, commit messages, PR bodies, or chat. This is a hard rule, not a preference. Do not substitute a hyphen, en dash, or other dash either. If a sentence seems to need one, rephrase it: split it into two sentences, or use a comma, colon, parentheses, or plain words so it reads naturally without any dash. The only `—` allowed in this codebase is the lone empty-name placeholder glyph (`name || '—'`) shown when a category or section has no name; that is a UI glyph, not punctuation.

## Data model: get this right

Hierarchy: **Collection › Category › Section › Entry.**
- **Category**: a colored group whose name is the card's *eyebrow*.
- **Section**: its name is the card *title*.
- **Entry**: a command plus description plus optional question.

**A section becomes ONE OR MORE cards. It is NOT one section = one card.**
`packSection()` greedily packs a section's entries onto a card until they overflow the printable body, then starts another card. A single too-tall entry is forced alone; an empty section yields one empty card. Each card carries `page`/`total` (page N of M *within the section*, footed only when `total>1`) and `gidx`/`gtotal` (global index across the whole deck). When describing this anywhere (README, UI copy, comments), say "a section becomes a card; long sections spill onto extra cards," never "a card = a section."

Card **front** lists the section's commands; the **back** shows their questions.

**Deck**: `{id,name,secIds[]}`, a named subset of ONE collection's sections.

## Expansions are additive

There are 12 readymade collections in `READYMADE` (order matters for the picker): `linux` (base/SAMPLE), `ssh`, `shell` (bash+zsh), `http`, `arch` (archives & transfer), `toys`, `devops`, `ai`, `vim`, `tmux`, `git`, `text`. Each non-base expansion is a full collection merged ON TOP of the `linux` base. The base holds the fundamentals; an expansion holds only the depth the base lacks. So an expansion must contain nothing the base already covers. Strip every base-covered command, both literal and semantic (e.g. the base's Vim keycap `` `dd` `` equals an expansion's bare `dd`).

Invariant to preserve on any content edit:
- every expansion has **0 base-overlap**, and
- **base + any expansion = 0 duplicate commands on a card** (verify via the app's own `mergeCategories`).

When verifying, normalize case-SENSITIVELY (strip backticks, collapse whitespace, do NOT lowercase): curl flags like `-I` vs `-i` are different commands and lowercasing falsely flags them as duplicates.

Two cross-expansion rules:
- curl REQUEST depth lives in the `http` deck, not `devops`. DevOps keeps only jq/yq under a "JSON & YAML" category, so `http + devops` loads with 0 curl overlap.
- check a new deck against likely co-loaded decks too, not only the base (e.g. `ssh` and `arch` both touch rsync/scp).

Legitimate exception: the same command may appear once per *tool or context card* (e.g. `/clear` on each assistant's card in the AI deck, or `git add` in both staging and conflict-resolution). Those are different cards, not redundancy.

## Verifying changes

- JS syntax: `node --check <(sed -n '/<script>/,/</script>/p' index.html | sed '1d;$d')`
- Behavior: `python3 -m http.server` then drive it with Playwright (assert on the DOM / `getBoundingClientRect`; MCP screenshots are NOT written to the repo).
- Data-model or expansion edits: run the `make()` factories in Node and check the additive invariant above before committing.

## Conventions

- One task = one branch off `main`, then commit, push, PR. Commit trailer: `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>`.
- Smallest working diff. Mark deliberate simplifications with a `// ponytail:` comment.
- `cd` into the repo prints a harmless zoxide warning; commands still run.
- New-expansion PRs all edit the same two anchors (factory before `const READYMADE`, registry line after the `linux` row), so two open ones always conflict and stale once one merges. Merge expansion PRs one at a time, back to back, re-pulling `main` between them. Conflicts are positional only: resolve by union in branch order (each factory gets its own close, keep both registry lines), re-run the invariant, then `git push --force-with-lease`. GitHub's mergeable flag lags a force-push, so trust a local `git merge --no-commit --no-ff` dry-run against `origin/main`.
