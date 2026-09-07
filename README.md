# ACE03-GETCERT Certification Journey — Notes

Google Cloud Associate Cloud Engineer (ACE) study notes pipeline for the
"ACE03-GETCERT_2026.06.29 Certification Journey Program" cohort. Pulls Meet
auto-transcripts from Drive recordings, turns them into structured summaries,
then into interactive local-served HTML study pages.

## Layout

```
recordings/   raw transcripts, one per session (timestamp + text, deduped/sorted)
summaries/    per-session Markdown summaries + one consolidated exam-focus doc
web/          interactive local site (notebook + exam review + index)
```

- `summaries/ACE03_總複習與考試重點.md` — consolidated Traditional-Chinese
  review across sessions #2–#6, plus ACE exam focus/tips section.
- `web/index.html` — landing page linking to the two study pages.
- `web/notebook.html` — visual "field notebook" (session #2–#6 bullet cards)
  with a 12-question scored quiz.
- `web/exam-review.html` — exam-prep page: collapsible per-session recap,
  filterable concept map, flip-cards for easily-confused concepts.

Both `web/` pages support light/dark/system theme via a toggle button
(top-right, persisted in `localStorage`).

## Serving the web pages locally

Uses Python's built-in `http.server` inside the existing project venv — no
extra install needed.

```bash
source ~/Project/.venv/bin/activate
cd web
python3 -m http.server 8080
```

Then open:
- http://localhost:8080/index.html
- http://localhost:8080/notebook.html
- http://localhost:8080/exam-review.html

Stop the server with `Ctrl+C` (or `kill <pid>` if run in background).

## Pipeline for adding a new session (repeatable)

1. **OPEN** — open the Drive recording + transcript panel (Meet auto-transcript).
2. **FETCH** — chrome-in-claude: `tabs_context_mcp` → find open tab →
   `read_page(ref_id=<transcript sidebar>, max_chars=2000000)`
   (skip `get_page_text` — transcript lives in an overlay, not main content).
3. **PARSE** — run `extract.py` on the dump → `[timestamp] text`, sorted,
   deduped (regex must accept both `M:SS` and `H:MM:SS`).
4. **SAVE** — write raw transcript to `recordings/<date>_<session>_transcript.txt`.
5. **SUMMARIZE** — read full transcript, split by speaker/topic segments; pull
   out action items, rules/deadlines, technical concepts, Q&A →
   `summaries/<date>_<session>_summary.md`.
6. **NOTEBOOK** — fold the new session into `web/notebook.html` (bullet cards:
   icon + 3–5 line bullets; tabular/comparison data as real tables; extend the
   quiz array with a few new questions drawn only from the summary).
7. **PUBLISH** (optional) — Artifact tool, distinct title, favicon, share link
   back to the user, if a shareable cloud copy is wanted alongside the local
   pages.

Reusable pieces: `extract.py` (fixed regex, scratchpad) and `notebook.html` /
`exam-review.html` as templates — future sessions just need new content
swapped into the card grid + quiz array.
