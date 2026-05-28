# Claude Code Skills

A collection of custom skills for [Claude Code](https://claude.ai/claude-code).

---

## Skills

### `readwise-digest`

Generates a weekly editorial knowledge digest from Readwise highlights and publishes it directly to Readwise Reader.

The skill pulls the past 7 days of highlights, analyzes them for cross-source connections and knowledge gaps, finds real linked next reads via web search, and writes a formatted newsletter article saved to your Reader inbox.

**Trigger phrases**
- "run my weekly digest"
- "analyze my highlights"
- "what am I missing in my reading"
- "knowledge gaps from Readwise"
- "weekly digest"

**Requirements**
- Readwise MCP connected
- `WebSearch` tool available

**What it produces**

An article in your Readwise Reader inbox titled `Weekly Knowledge Digest — [Date]`, containing 3–5 editorial insights, each with a declarative headline, analytical prose, a named knowledge gap, and a verified link to a next read.

**Scheduling**

When triggered, the skill will offer to schedule itself to run automatically every Saturday at 9 AM via `mcp__scheduled-tasks`.

---

### `readwise-highlight-synthesis`

Analyzes Readwise highlights filtered by one or more tags, synthesizes generalizable insights across sources, and publishes the result as a formatted article to your Readwise Reader inbox.

The skill runs a two-phase workflow: it first retrieves highlights using both semantic (relevance-ranked) search and a recency supplement to ensure nothing recent is missed, then writes a structured analytical essay organized into **Strong Convergences**, **Productive Tensions & Discrepancies**, and **Structural Gaps**. The final article is saved to your Reader inbox with standardized metadata so you can build a filtered view of all Claude-generated analyses over time.

**Trigger phrases**
- "analyze my highlights tagged [topic]"
- "what do my highlights say about [topic]"
- "synthesize highlights on [topic]"
- "find insights across my [topic] reading"
- Any request combining Readwise + a topic + an action like export, analyze, or summarize

**Requirements**
- Readwise MCP connected (`readwise_search_highlights`, `reader_create_document`)

**What it produces**

An article in your Readwise Reader inbox titled `Readwise Highlight Analysis — YYYY-MM-DD HH:MM UTC`, containing:
- A TL;DR blockquote
- **I. Strong Convergences** — where multiple sources independently land on the same claim
- **II. Productive Tensions & Discrepancies** — where sources disagree or hold incompatible assumptions
- **III. Structural Gaps** — what the corpus collectively fails to address or underweights

All output documents are tagged with the queried tags plus the permanent workflow marker `claude-highlight-synthesis`.

**Notes on retrieval**
- The skill merges semantic search (relevance-ranked, up to 50 results) with a recency supplement via `reader_list_documents` to avoid underrepresenting new additions
- When multiple tags are requested, only highlights matching **all** tags are included
- If fewer than 5 highlights qualify, the synthesis notes the thin corpus and caveats confidence accordingly

---

## Installation

1. Clone this repo into your Claude Code skills directory:

   ```bash
   git clone https://github.com/Felipe-Furtado/skills.git ~/.claude/skills
   ```

2. Register the skills directory in your Claude Code settings (`~/.claude/settings.json`):

   ```json
   {
     "skillsPath": "~/.claude/skills"
   }
   ```

3. Restart Claude Code. Skills will be available immediately.

---

## Structure

```
skills/
├── readwise-digest/
│   └── SKILL.md                # Weekly knowledge digest workflow
├── readwise-highlight-synthesis/
│   └── SKILL.md                # Tag-filtered highlight analysis & synthesis
└── README.md
```

Each skill lives in its own folder with a `SKILL.md` file that contains the frontmatter metadata (name, description, trigger hints) and the full workflow instructions Claude follows when the skill is invoked.
