# claude-skills

My hand-written skills for Claude (Claude Code / Cowork). Each skill lives in
`skills/<name>/SKILL.md`.

## Skills

- **[skill-scout](skills/skill-scout/SKILL.md)** — evaluate, critique, select, and
  audit AI agent skills themselves. Vets a `SKILL.md` the way you'd vet a
  dependency: judge from the task and the content, not from the tech stack. Modes:
  critique a skill, decide install-or-not, find skills for a task, audit installed
  skills, and proactively flag when a skill would help.

## Install

### Everything, via the skills CLI
```bash
npx skills@latest add <your-username>/claude-skills
```

### One skill, manually (global — available in every project)
```bash
git clone https://github.com/<your-username>/claude-skills.git /tmp/claude-skills
mkdir -p ~/.claude/skills
cp -r /tmp/claude-skills/skills/skill-scout ~/.claude/skills/
```

### One skill, per-project
Copy the folder into the project's `.claude/skills/` instead of `~/.claude/skills/`.

Verify with `ls ~/.claude/skills/` — each skill is a folder containing `SKILL.md`.

## License

MIT
