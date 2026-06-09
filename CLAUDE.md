# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

A Claude Code skills plugin package (`marketingskills`) by Corey Haines — 33 marketing skills for technical marketers and founders. Skills are invokable AI agent behaviors (CRO, copywriting, SEO, paid ads, etc.) distributed via the `npx skills add` mechanism.

## Repository Structure

```
.claude-plugin/marketplace.json   # Plugin manifest — entry point for `npx skills add`
.claude/skills/<skill-name>/
  SKILL.md                        # Skill definition (YAML frontmatter + markdown instructions)
  evals/evals.json                # Test cases for the skill
  references/<name>.md            # Optional supporting reference documents
```

There is no build toolchain, package.json, or test runner. All content is markdown and JSON.

## Skill File Format

Every `SKILL.md` uses YAML frontmatter followed by markdown:

```markdown
---
name: skill-name
description: Trigger conditions — "Also use when the user mentions X, Y, Z..."
metadata:
  version: 1.1.0
---

# Skill Title
...
```

The `description` field is the routing signal: it determines when Claude selects this skill. It must cover both explicit invocation (`/skill-name`) and natural-language trigger phrases. Current version convention: `1.1.0` for all updated skills; increment the minor version on substantive changes.

## Key Architectural Pattern: product-marketing-context

`product-marketing-context` is the foundational skill. It creates `.agents/product-marketing-context.md` (or `.claude/product-marketing-context.md` in older setups) — a persistent document capturing product positioning, audience, pain points, and brand voice.

**Every other skill checks for this file first** before asking the user questions. When adding or editing skills, include this block at the top of the "Before Writing" / "Initial Assessment" section:

```markdown
**Check for product marketing context first:**
If `.agents/product-marketing-context.md` exists (or `.claude/product-marketing-context.md` in older setups), read it before asking questions. Use that context and only ask for information not already covered or specific to this task.
```

## Adding a New Skill

1. Create `.claude/skills/<skill-name>/` with `SKILL.md`, `evals/evals.json`, and optionally `references/`
2. Register it in `.claude-plugin/marketplace.json` under `plugins[0].skills` as `"./skills/<skill-name>"`
3. `SKILL.md` should follow the existing structure: initial assessment → core principles → frameworks → output format → related skills
4. `evals/evals.json` format:
   ```json
   {
     "skill_name": "skill-name",
     "evals": [
       {
         "id": 1,
         "prompt": "...",
         "expected_output": "Prose description of what the skill should do",
         "assertions": [
           "Checks for product-marketing-context.md",
           "Does X",
           "Does Y"
         ],
         "files": []
       }
     ]
   }
   ```
   Assertions are behavioral checks written as short declarative strings. `files` is an array of file paths to pre-populate in the eval context (usually empty).
5. Reference documents go in `references/` and are linked from `SKILL.md` with relative paths like `[references/guide.md](references/guide.md)`

## Cross-Skill References

Skills link to each other in a "Related Skills" section at the bottom using bullet format:
```markdown
## Related Skills
- **other-skill**: One-line description of when to use it instead
```

Skill boundary rules matter: e.g., `copywriting` writes new copy, `copy-editing` polishes existing copy, `page-cro` handles page structure strategy. When editing a skill's description or related-skills section, check the referenced skills to ensure the routing logic stays consistent.

## Plugin Manifest

`.claude-plugin/marketplace.json` is the entry point for `npx skills add`. Key fields:

- `plugins[0].source`: `"./"` — base path for skill resolution; skill paths like `"./skills/ab-test-setup"` resolve relative to this
- `plugins[0].strict`: `false` — skills can trigger on natural language, not only explicit `/skill-name` invocation
- `plugins[0].skills`: ordered array of skill paths; add new skills here when creating them
