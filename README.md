# Redline

A copy-editor's desk for text, in the browser. Dark manuscript page,
red-ink markup, no sign-up, nothing leaves your device except the two
CDN libraries it loads for file import.

**Live artifact:** https://claude.ai/artifact/KZEFHEvZErSN6r2ZnEThCA

## What it does

| Mode | What you get |
|---|---|
| **paraphrase** | Swaps words via a built-in synonym set. Struck-through = original, italic = replacement. Three intensities: standard / fluency / creative. |
| **humanize** | Paraphrases, then varies sentence rhythm (merges short sentences, random openers) and applies contractions, to read less uniform. Light / standard / heavy. |
| **summarize** | Extractive summary — scores sentences by keyword weight, keeps the top-scoring ones in original order. Short / medium / long keeps ~25% / 40% / 60% of sentences. |
| **stats** | Word count, sentence count, avg words/sentence, reading time, and a Flesch reading-ease score with a plain-language difficulty label. |
| **read-check** | A heuristic 0–100 "AI-likelihood" score from sentence-length uniformity (burstiness), vocabulary diversity, and stock AI phrase hits ("delve into", "it's important to note", etc). |

Plus: **import** `.txt` / `.docx` / `.pdf` straight into the editor, and
**download** any result as a `.txt` file.

## Running it

It's a single self-contained `.html` file. Two ways to use it:

- Open the published artifact link above, or
- Download `redline.html` and open it directly in any browser — no
  server, no build step, no install.

Two libraries load from CDN for file import (everything else is
inline, zero dependencies):
- [`pdf.js`](https://mozilla.github.io/pdf.js/) (3.11.174) — extracts
  text from `.pdf`
- [`mammoth.js`](https://github.com/mwilliamson/mammoth.js) (1.6.0) —
  extracts text from `.docx`

If you're offline, paraphrase/humanize/summarize/stats/read-check all
still work — only file import needs the CDN.

## How it works

Everything runs client-side in plain JavaScript — no backend, no API
calls, no data sent anywhere:

- **paraphrase** — a ~25-word built-in synonym dictionary (`SYN`),
  matched case-insensitively per word with punctuation preserved, swapped
  in at a probability set by the chosen intensity.
- **humanize** — runs paraphrase first, then sentence-level
  rewrites (occasional sentence merges, random openers like "Honestly,")
  and a contraction pass (`do not` → `don't`, etc).
- **summarize** — term-frequency sentence scoring against a small
  stopword list, keeps the top N sentences by score, re-sorted back into
  original order.
- **stats** — standard Flesch Reading Ease formula
  (`206.835 − 1.015×(words/sentences) − 84.6×(syllables/words)`) with a
  regex-based syllable estimator (vowel-group counting).
- **read-check** — a weighted heuristic: `(1−burstiness)×45 +
  (1−vocab_diversity)×35 + stock_phrase_density×20`, where burstiness is
  the coefficient of variation of sentence lengths.
- **import** — `.txt` via `FileReader`, `.docx` via mammoth's
  `extractRawText`, `.pdf` via pdf.js page-by-page text extraction.

## Honest limitations

- **Paraphrase coverage is narrow.** The synonym set is a few dozen
  common words, not a full thesaurus — good for a demo, not comprehensive.
  The standalone `paraphraser.py` script (WordNet-backed) has much wider
  coverage if you need that.
- **read-check is a heuristic, not a certified AI detector.** It's built
  from the same *categories* of signal real detectors use, but it's not
  trained on a labeled dataset. Treat the score as a rough signal.
- **No plagiarism checking.** That needs a live index of the web/papers
  to compare against — not something a static page can carry. Nothing
  here claims to do this.
- **humanize can break factual/technical precision.** Synonym swaps
  don't understand meaning — review anything before using it for work
  that needs to stay accurate.
- Using this to get past a required-original-work check (school,
  publishing, etc.) is worth thinking twice about — most places treat
  that as an integrity issue regardless of whether a detector catches it.

## File

- `redline.html` — the entire app, one file, ~15KB.
