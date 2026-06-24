# TermDecks — working notes

Single-file web app (`index.html`, vanilla JS + CSS + HTML, no build, no deps). It builds printable, poker-sized (2.5″ × 3.5″) terminal-command reference cards, with an on-screen preview, a 3×3 print sheet, and a Leitner spaced-repetition quiz. State lives in `localStorage` key `cardcheat_v3`.

## Data model — get this right

Hierarchy: **Collection → Category → Section → Entry.**
- **Category** — a colored group; its name is the card's *eyebrow*.
- **Section** — its name is the card *title*.
- **Entry** — a command + description + optional question.

**A section becomes ONE OR MORE cards. It is NOT one section = one card.**
`packSection()` greedily packs a section's entries onto a card until they overflow the printable body, then starts another card; a single too-tall entry is forced alone; an empty section yields one empty card. Each card carries `page`/`total` (page N of M *within the section*, footed only when `total>1`) and `gidx`/`gtotal` (global index across the whole deck). When describing this anywhere — README, UI copy, comments — say "**a section becomes a card; long sections spill onto extra cards**," never "a card = a section."

Card **front** = the section's commands; **back** = their questions.

**Deck** = `{id,name,secIds[]}` — a named subset of ONE collection's sections.

## Expansions are additive

Each expansion (`toys, devops, ai, vim, tmux, git, text`) is a full collection merged ON TOP of the base starter deck (the `linux`/SAMPLE collection). The base holds the fundamentals; an expansion holds only DEPTH the base lacks. So an expansion must contain **nothing the base already covers** — stripped of every base-covered command, both literal and semantic (e.g. the base's Vim keycap `` `dd` `` equals an expansion's bare `dd`).

Invariant to preserve on any content edit:
- every expansion has **0 base-overlap**, and
- **base + any expansion = 0 duplicate commands on a card** (verify via the app's own `mergeCategories`).

Legitimate exception: the same command may appear once per *tool/context card* (e.g. `/clear` on each assistant's card in the AI deck, or `git add` in both staging and conflict-resolution) — those are different cards, not redundancy.

## Verifying changes

- JS syntax: `node --check <(sed -n '/<script>/,/</script>/p' index.html | sed '1d;$d')`
- Behavior: `python3 -m http.server` then drive it with Playwright (assert on the DOM / `getBoundingClientRect`; MCP screenshots are NOT written to the repo).
- Data-model / expansion edits: run the `make()` factories in Node and check the additive invariant above before committing.

## Conventions

- One task = one branch off `main` → commit → push → PR. Commit trailer: `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>`.
- Smallest working diff. Mark deliberate simplifications with a `// ponytail:` comment.
- `cd` into the repo prints a harmless zoxide warning; commands still run.
