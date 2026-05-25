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
│   └── SKILL.md        # Skill definition and workflow
└── README.md
```

Each skill lives in its own folder with a `SKILL.md` file that contains the frontmatter metadata (name, description, trigger hints) and the full workflow instructions Claude follows when the skill is invoked.
