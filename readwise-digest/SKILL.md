---
name: readwise-digest
description: >
  Generates a weekly editorial knowledge digest from Readwise highlights and
  publishes it to Readwise Reader. Use this skill whenever the user wants to
  analyze their recent highlights, surface knowledge gaps, find missing
  connections across their reading, or produce a weekly digest. Trigger on
  phrases like "weekly digest", "analyze my highlights", "what am I missing in
  my reading", "run my reading digest", "knowledge gaps from Readwise", or any
  request to reflect on or synthesize recent reading activity. Requires the
  Readwise MCP to be connected. Offer to schedule it weekly if the user doesn't
  ask first.
---

# Weekly Knowledge Digest

Pulls recent Readwise highlights, finds cross-source connections and knowledge
gaps, locates real linked next reads, and publishes an editorial newsletter
directly to Readwise Reader.

**Requirements**: Readwise MCP connected · WebSearch available

---

## Workflow

### 1 · Pull highlights

Call `readwise_list_highlights` with:
- `highlighted_at_gt`: 7 days before today (ISO 8601)
- `page_size`: 200
- `response_fields`: `["text", "note", "highlighted_at", "book_title", "book_author", "book_category", "tags"]`

Stop early and tell the user if fewer than 5 highlights are returned — not
enough material for a meaningful digest.

### 2 · Analyze

Group highlights by source. Look for three things:

- **Cross-source connections** — two unrelated sources circling the same
  underlying question without referencing each other
- **Knowledge gaps** — an important question a source raises but doesn't answer
- **Missing domains** — adjacent fields entirely absent from the week's reading
  that would reframe what's there

Select 3–5 of the most intellectually interesting combinations. Cross-domain
connections are more valuable than single-source observations.

### 3 · Find real next reads

For each insight, use WebSearch to find one specific, linkable piece of content
that directly addresses the gap. Prefer substantive journalism, research
institutions, or practitioner writing over listicles. Verify the URL is real
before including it.

### 4 · Write the newsletter

**Voice**: Editorial and reader-agnostic — like a sharp editor who read
everything and found where it connects. Never address the reader as "you/your";
use "the reading", "this week's highlights", "the argument" instead. Write in
paragraphs, not lists.

Each insight needs:
- A punchy **declarative headline** (not a question, not a summary)
- **2–3 short paragraphs** of editorial analysis that earn the headline
- A **Gap callout** — 1–2 sentences naming the specific knowledge gap
- A **Read this next** link with a one-sentence description of why it closes
  the gap

### 5 · Publish to Readwise Reader

Call `reader_create_document` with:

```
title:          "Weekly Knowledge Digest — [Month D, YYYY]"
html:           full newsletter HTML (see template below)
url:            "https://weekly-digest.internal/[YYYY-MM-DD]"
category:       "article"
tags:           ["weekly-digest", "knowledge-gaps"]
published_date: today in ISO 8601
summary:        one sentence covering the main themes this week
```

The document lands in the user's Reader inbox. Readwise push/email
notifications handle delivery from there.

---

## HTML Template

Readwise Reader strips all CSS — `<style>` blocks and inline styles are
discarded. Use semantic HTML only; Reader applies its own reading styles.

Structure the document as follows:

```html
<article>

  <h1>Weekly Knowledge Digest — [Month D, YYYY]</h1>
  <p><em>[N] highlights · Week of [date range]</em></p>
  <p>[2–3 sentence intro]</p>

  <hr>

  <h2>[Insight headline]</h2>
  <p><em>Highlights from: [Source Title A], [Source Title B]</em></p>

  <p>[Body paragraph 1]</p>
  <p>[Body paragraph 2]</p>
  <p>[Body paragraph 3, if needed]</p>

  <blockquote>
    <p><strong>Gap</strong></p>
    <p>[1–2 sentences naming the specific knowledge gap]</p>
  </blockquote>

  <p><strong>Read This Next</strong></p>
  <p><a href="[url]">[Article title — Publication]</a></p>
  <p><em>[One sentence: why this closes the gap]</em></p>

  <hr>

  [Repeat for each insight]

  <h3>Sources This Week</h3>
  <ul>
    <li>[Source title — Author]</li>
    <li>[Source title — Author]</li>
    ...
  </ul>

</article>
```

Key rules:
- `<hr>` between every insight — this is the only reliable separator in Reader
- Source attribution always uses "Highlights from:" prefix, comma-separated titles
- `<blockquote>` for the Gap callout — Reader renders it with a left indent/border
- "Read This Next" on its own `<p>`, link on the next `<p>`, description on the next
- Footer sources as `<ul>` bullet list, not inline prose

---

## Scheduling

If the user wants this to run automatically, offer to create a weekly scheduled
task with `mcp__scheduled-tasks__create_scheduled_task`:
- `cronExpression`: `"0 9 * * 6"` (Saturdays at 9 AM local time)
- Prompt: the full workflow above, self-contained
