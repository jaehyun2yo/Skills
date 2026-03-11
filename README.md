# Skills

Personal skill collection for Claude Code.

## Usage

Each skill is a self-contained folder with a `SKILL.md` file and optional supporting files.

To use a skill, copy or symlink the folder into `~/.claude/skills/` or reference it in your CLAUDE.md.

## Skills

| Skill | Description |
|-------|-------------|
| [auditing-project-readiness](auditing-project-readiness/) | Use when starting a new project, onboarding to an existing codebase, or when development workflow feels stuck |

## Structure

```
skill-name/
  SKILL.md              # Main skill file (required)
  supporting-file.*     # Only if referenced from SKILL.md
```

- Folder name = frontmatter `name`
- Flat namespace, no nested categories
