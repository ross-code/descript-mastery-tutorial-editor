# Descript Mastery — Tutorial Editor (Claude Skill)

A Claude [Agent Skill](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview)
for editing **Descript Mastery** tutorial videos end to end.

Two halves meet on the Descript timeline:

- **Descript** carries the editorial backbone — transcribe, strip fillers, tighten,
  Studio Sound, captions, publish — driven through the Descript MCP in small, verifiable passes.
- **HyperFrames** carries the branded graphics layer — intro, "Coming Up" roadmap, lower third,
  outro, subscribe cards, and on-footage "liquid glass" overlays — rendered from the existing
  Descript Mastery brand kit and composited onto the footage.

The skill follows a three-gate, human-in-the-loop workflow (rough cut → graphics plan →
composited cut) and is grounded in the real brand system (`design.md`), the built kits
(`brand/`, `brand-vertical/`), and the documented HyperFrames render gotchas (`HANDOFF.md`).

## Install

**Claude Desktop / Cowork:** install the packaged `.skill` bundle via the *Save skill* button,
or add it under **Settings → Capabilities**.

**Claude Code:** copy the `descript-mastery-tutorial-editor/` folder into your skills directory
(e.g. `~/.claude/skills/`).

## Contents

```
descript-mastery-tutorial-editor/
  SKILL.md      # the skill definition (frontmatter + instructions)
```

## Notes

- Body font assumed: **Hanken Grotesk** (Google Fonts, render-safe). Switzer `.woff2` files are
  bundled in the brand kit if you want a pixel-exact swap.
- The skill reads brand truth from disk — it does not hard-code or invent brand values.
- Brand isolation: never blend Descript Mastery with any other channel's or project's assets/colors/fonts.
