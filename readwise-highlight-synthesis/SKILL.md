---
name: readwise-highlight-synthesis
description: >
  Analyze Readwise highlights filtered by one or more tags, synthesize generalizable insights
  across sources, and publish the analysis as a formatted article to Readwise Reader inbox.
  Use this skill whenever the user asks to: analyze highlights by tag(s), find patterns or
  tensions across their reading on a topic, synthesize what they've highlighted about a
  subject, or export a highlight analysis to Reader. Trigger phrases include "analyze my
  highlights tagged", "what do my highlights say about", "synthesize highlights on",
  "find insights across my [topic] reading", or any request combining Readwise + a topic +
  an action like export, analyze, or summarize.
compatibility:
  required_mcps:
    - Readwise (readwise_search_highlights, reader_create_document)
---

# Readwise Highlight Synthesis

A two-phase workflow: (1) retrieve and analyze highlights filtered by tag(s), then (2) publish
a structured synthesis article to the user's Readwise Reader inbox.

---

## Phase 1: Retrieve Highlights

### 1a. Semantic search (relevance-ranked)

Use `readwise_search_highlights` with:
- `vector_search_term`: a rich semantic query capturing the topic (e.g. "AI artificial intelligence labor market work jobs employment")
- `full_text_queries`: one entry per requested tag, each with `field_name: "highlight_tags"`
- `limit: 50` — this is the **hard API maximum**, not a design choice; always use 50

```
readwise_search_highlights(
  vector_search_term="<rich semantic description of topic>",
  full_text_queries=[
    { field_name: "highlight_tags", search_term: "<tag1>" },
    { field_name: "highlight_tags", search_term: "<tag2>" }
  ],
  limit=50
)
```

**Important limitation:** `readwise_search_highlights` has no sort parameter — results are
ranked by relevance score only, with no chronological option. This means a large and growing
highlight library can have recent additions underrepresented if they score lower on semantic
similarity.

### 1b. Recency supplement (chronological)

To ensure recent highlights are captured, also call `reader_list_documents` for each
requested tag, sorted by recency, and merge any qualifying highlights not already present
in the semantic results:

```
reader_list_documents(
  category="highlight",
  tag=["<tag1>", "<tag2>"],   # pass all tags to get AND filtering
  limit=50
)
```

Sort the merged pool by `updated_at` descending before analysis, so the synthesis naturally
weights recent material. Deduplicate by highlight ID before proceeding.

**Filtering:** After retrieval from both sources, manually filter to only highlights whose
`highlight_tags` array contains **all** requested tags. Do not include highlights that match
only one tag when two are requested.

---

## Phase 2: Synthesize Analysis

Organize the qualifying highlights into a structured analytical essay with three sections:

### 2a. Areas of Strong Convergence
Where multiple sources independently land on the same claim. Group by theme, not by source.
Name sources inline (e.g. "NFX", "Mollick", "McKinsey") without a trailing source map.

### 2b. Productive Tensions and Discrepancies
Where sources disagree, contradict, or hold incompatible assumptions. Explain what's at stake
in the disagreement — why it matters for policy, investment, or practice.

### 2c. Structural Gaps
What the corpus collectively fails to address, underweights, or only gestures at. These are
research or reading recommendations, not just summaries of absence.

### Citation and quoting conventions
- Attribute all claims to their source inline: **Source Name — *Title*** or just **Source Name**
- Use *italicized quotation marks* for verbatim quotes: *"exact words from the highlight"*
- No source map or reference list at the end of the document
- Keep quotes purposeful — prefer paraphrase unless the exact wording is load-bearing

---

## Phase 3: Publish to Readwise Reader

Use `reader_create_document` with the following **standardized conventions** (non-negotiable
for reproducibility):

| Field | Value |
|---|---|
| `title` | `Readwise Highlight Analysis — YYYY-MM-DD HH:MM UTC` |
| `author` | `Claude via Readwise MCP` |
| `tags` | Queried tags (e.g. `ai`, `labor-market`) **+** `claude-highlight-synthesis` |
| `category` | `article` |
| `location` | Inbox (`new`) — default, no override needed |
| `published_date` | Current datetime in ISO 8601 |
| `summary` | 2–3 sentence abstract of findings |
| `url` | `https://claude.ai#<topic-slug>-<YYYYMMDD-HHMM>` (synthetic, for deduplication) |

### HTML formatting rules

Write `html` as clean, escaped HTML. **Never use CDATA wrappers** — they cause blank renders
in Reader. Escape special characters with HTML entities (`&mdash;`, `&times;`, `&rsquo;`,
`&ldquo;`, `&rdquo;`, `&ndash;`, etc.).

Structure:
```html
<article>
  <h1>[Topic] — Synthesis of Highlights</h1>
  <p><em>Generated YYYY-MM-DD HH:MM UTC · N highlights tagged X + Y · Sources: ...</em></p>
  <hr>
  <blockquote><p><strong>The short version:</strong> [1–2 sentence TL;DR]</p></blockquote>
  <hr>
  <h2>I. Strong Convergences</h2>
    <h3>1. ...</h3>
    ...
  <h2>II. Productive Tensions and Discrepancies</h2>
    <h3>N. ...</h3>
    ...
  <h2>III. Structural Gaps in the Corpus</h2>
    <h3>N. ...</h3>
    ...
  <hr>
  <p><em>Analysis generated by Claude via Readwise MCP on YYYY-MM-DD HH:MM UTC, from N highlights tagged "X" and "Y".</em></p>
</article>
```

Reader rendering tips:
- `<blockquote>` renders visually distinct — use for TL;DR and key verbatim quotes
- `<h2>` / `<h3>` hierarchy activates Reader's outline view and progress bar
- `<table>` renders cleanly — good for structured comparisons
- `<hr>` is a clean section separator; use sparingly
- Do not use inline styles

---

## Error handling

- If no highlights match **all** requested tags: tell the user, show how many matched each
  tag individually, and ask whether to proceed with a union (OR) or a single-tag filter.
- If fewer than 5 qualifying highlights: note the thin corpus in the synthesis intro and
  caveat confidence accordingly.
- If `reader_create_document` returns no `id`: the document did not save. Retry once with
  a fresh synthetic URL to avoid deduplication collision.

---

## Workflow tag

`claude-highlight-synthesis` is the permanent workflow marker tag on all output documents.
It enables the user to build a filtered Reader view of all Claude-generated cross-tag
analyses over time.
