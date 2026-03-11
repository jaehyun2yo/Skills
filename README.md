# Skills

Personal skill collection for Claude Code.

## Usage

### Install

Copy or symlink skill folders into `~/.claude/skills/`:

```bash
# Symlink (recommended — stays in sync)
ln -s /path/to/Skills/project-setup ~/.claude/skills/project-setup

# Or copy
cp -r /path/to/Skills/project-setup ~/.claude/skills/project-setup
```

### Trigger

Skills are invoked automatically by keyword or manually via slash command:

```
# Automatic — type trigger keywords in conversation
"셋업해줘" → project-setup
"프로젝트 진단" → auditing-project-readiness

# Manual — slash command
/project-setup
/auditing-project-readiness
```

## Skills

| Skill | Triggers | Description |
|-------|----------|-------------|
| [project-setup](project-setup/) | setup, 셋업, 셋팅, 설정, 프로젝트 설정, init setup | Scan → Score → Generate missing files with approval |

## Structure

```
skill-name/
  SKILL.md              # Main skill file (required)
  supporting-file.*     # Only if referenced from SKILL.md
```

- Folder name = frontmatter `name`
- Flat namespace, no nested categories
