# Skills

Personal skill collection for Claude Code — with Agent Team orchestration support.

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
"시작하자" → dev-start
"마무리하자" → dev-wrap

# Manual — slash command
/project-setup
/dev-start
```

## Skills

| Skill | Triggers | Description |
|-------|----------|-------------|
| [project-setup](project-setup/) | setup, 셋업, 셋팅, 설정, 프로젝트 설정, init setup | Scan → Score (A-K, 44pts) → Generate missing files with approval. Includes Agent Team readiness scoring |
| [skill-register](skill-register/) | 스킬 등록, register skill, 스킬 연결, 등록해줘 | Register skills to project — symlinks, CLAUDE.md triggers, verify |
| [usage-guide](usage-guide/) | 사용법, usage guide, 가이드 작성, 매뉴얼, how to use, 사용 문서 | Generate unified usage doc covering skills, plugins, hooks, agents, and Agent Team workflows |
| [dev-start](dev-start/) | 개발 시작, 코딩 시작, 시작하자, start coding, start dev, 작업 시작, 뭐하고 있었지, 이어서 개발, 어디까지 했지 | Session init — progress, git state, active team detection, complex task → team suggestion |
| [dev-wrap](dev-wrap/) | 마무리, wrap up, 세션 정리, 오늘 여기까지, 커밋하고 끝, 세션 끝, 마무리하자 | Session wrap-up — team cleanup, commit, update docs, handoff prep (also auto-triggers on Stop hook) |

## Agent Team Integration

All skills are Agent Team-aware:

- **dev-start**: Detects active teams, suggests team composition for complex tasks (parallel impl, multi-perspective review, adversarial debug)
- **dev-wrap**: Handles graceful team shutdown (teammates → cleanup), records team work in progress.txt
- **project-setup**: Scores Agent Team readiness (category K), generates team config templates, directory ownership maps, spawn prompt patterns
- **usage-guide**: Documents team workflows, composition patterns, cost optimization, session management

### Prerequisites

Enable Agent Teams in `settings.json`:
```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

## Structure

```
skill-name/
  SKILL.md              # Main skill file (required)
  supporting-file.*     # Only if referenced from SKILL.md
```

- Folder name = frontmatter `name`
- Flat namespace, no nested categories
