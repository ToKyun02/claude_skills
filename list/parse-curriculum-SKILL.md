---
name: parse-curriculum
description: Parse messy scraped/innerText curriculum or roadmap text (pasted from a browser, PDF, or site export) into clean, structured, hierarchical Markdown. Trigger when the user pastes raw scraped text and asks to clean it up, format it, turn it into markdown, or organize it into a roadmap/checklist.
---

# Parse Curriculum / Roadmap Text into Markdown

## When this triggers
The user pastes raw text copied from a webpage (e.g. via `innerText`, browser dev tools,
or a screen scrape) that represents a learning roadmap, curriculum, syllabus, or course
outline. The text usually has no reliable structure: indentation is lost, headings and
items run together, and there may be noise (nav links, ads, "Sponsored", cookie banners,
"Share this roadmap" buttons, etc).

## Goal
Turn that raw blob into clean, hierarchical Markdown that mirrors the original
information architecture — not just reformatted text, but correctly *re-inferred* structure.

## Process

1. **Strip noise first.**
   Remove anything that is clearly UI chrome, not curriculum content: nav bars, "Login",
   "Share", "Download PDF", ad copy, cookie/consent text, repeated site header/footer
   text, social share counts, breadcrumbs. When unsure whether a line is noise or a real
   topic, keep it and flag it rather than silently dropping it — list flagged lines at
   the end under "Uncertain / please verify".

2. **Re-infer the hierarchy.**
   innerText scraping flattens nesting. To reconstruct levels, use these signals in
   order of reliability:
   - Explicit numbering or lettering in the source (e.g. "1.", "1.1", "a)") — strongest signal
   - Repeated category words the site uses for section headers (e.g. "Beginner", "Intermediate",
     "Advanced", role names, quarter/module numbers)
   - Capitalization patterns (ALL-CAPS or Title Case short lines are usually section headers;
     longer sentence-case lines are usually leaf topics or descriptions)
   - Known domain knowledge — if this is a recognizable roadmap (e.g. a common tech
     curriculum), use standard groupings for that field as a sanity check, but never
     invent topics that aren't in the source text
   - Line length and position — a short line immediately followed by several longer,
     related lines is very likely a parent with children

3. **Build the Markdown.**
   - `#` for the roadmap/curriculum title
   - `##` for top-level sections/phases (e.g. "Beginner", "Module 1")
   - `###` for sub-topics within a section
   - `- [ ]` checkbox list items for individual, actionable learning items (leaf nodes)
   - Preserve any resource links or references verbatim if present in the source
   - Do not paraphrase or summarize topic names — keep the source's own wording for
     topic titles; only clean up whitespace/casing artifacts from the scrape

4. **Deduplicate.**
   Scraped pages often repeat the same node text twice (once for the diagram node, once
   for a tooltip/description). Merge duplicates; if one version has more detail, keep
   the detailed one.

5. **Output format.**
   Produce a single Markdown document:
   ```markdown
   # <Roadmap Title>

   ## <Section 1>
   - [ ] Topic
   - [ ] Topic
     - [ ] Sub-topic (if the source clearly nests it)

   ## <Section 2>
   ...

   ---
   ## Uncertain / please verify
   - <any line that was ambiguous — is it noise, a topic, or a header?>
   ```

6. **If the user has an existing checklist file** (e.g. `LEARNING.md` from a prior sync),
   merge instead of overwrite: keep any `- [x]` (completed) items checked if the topic
   text matches or closely matches an item in the new parse, and only append genuinely
   new topics.

## Notes
- Never fetch the source page yourself in this skill — this skill only operates on text
  the user has already pasted or saved to a file. If the user wants the live page
  fetched and parsed, that's the `sync-roadmap` skill's job instead.
- Ask the user for the roadmap's title/source if it isn't obvious from the pasted text,
  rather than guessing.
- Keep output terse — this is a reference checklist, not prose. No added commentary
  inside the Markdown body itself.
