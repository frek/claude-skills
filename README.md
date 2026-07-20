# claude-skills

My hand-written skills for Claude (Claude Code / Cowork). Each skill lives in
`skills/<name>/SKILL.md`.

## Skills

- **[skill-scout](skills/skill-scout/SKILL.md)** — evaluate, critique, select, and
  audit AI agent skills themselves. Vets a `SKILL.md` the way you'd vet a
  dependency: judge from the task and the content, not from the tech stack. Modes:
  critique a skill, decide install-or-not, find skills for a task, audit installed
  skills, and proactively flag when a skill would help.
- **[figma-senior-mode](skills/figma-senior-mode/SKILL.md)** — a senior-mindset
  overlay for Figma work over MCP (complements the official `figma-*` skills).
  Forces a design-system preflight (Token Map + Component Registry), deliberate
  hug/fill, states & edge cases, accessibility, and clarifying questions *before*
  drawing instead of rework after.

## Install

### Everything, via the skills CLI
```bash
npx skills@latest add frek/claude-skills
```

### Manually (global — available in every project)
```bash
git clone https://github.com/frek/claude-skills.git /tmp/claude-skills
mkdir -p ~/.claude/skills
cp -r /tmp/claude-skills/skills/skill-scout       ~/.claude/skills/
cp -r /tmp/claude-skills/skills/figma-senior-mode ~/.claude/skills/
```

### Per-project
Copy the folder into the project's `.claude/skills/` instead of `~/.claude/skills/`.

Verify with `ls ~/.claude/skills/` — each skill is a folder containing `SKILL.md`.

## Maintenance

`figma-senior-mode` cross-references the official `figma-use` skill by rule number
(`§1 rule 8 — fonts`, etc.). Those numbers can drift when Figma updates its skills.
When you update the Figma MCP plugin, re-verify these four still point at the right rule:

| In figma-senior-mode | Should match figma-use |
|---|---|
| §1 rule 3a | `getPluginData`/`setPluginData` unsupported → `getSharedPluginData` |
| §1 rule 8  | text-edit recipe / `listAvailableFontsAsync` ("Semi Bold", not "SemiBold") |
| §1 rule 9  | `setCurrentPageAsync` (sync `figma.currentPage =` throws) |
| §1 rule 15 | return all created/mutated node IDs |

Verified against the installed `figma-use` on 2026-07-20.

## License

MIT
