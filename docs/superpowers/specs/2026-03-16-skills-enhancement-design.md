# Skills Enhancement Design — 2026-03-16

## Summary

Enhance the existing Skills project with 6 improvements based on 2026 Claude Code ecosystem trends:
- A–B: Expand project-setup scoring & templates (advanced hooks)
- C: Add self-improving pattern to code-polish
- D: Optimize Progressive Disclosure across all skills
- E: New skill — skill-eval (eval/benchmark runner)
- F: New skill — security-guard (automated security prevention)

## Approach: Layered Extension (Option C)

Existing skill structure preserved. Each improvement is an independent additive layer.
No breaking changes to current workflows.

---

## A. checklist.md — Hooks Category Expansion

### Current State
- Category B: Hooks, 6 items (B1–B6), 6 points total
- Total score: 46 points across 11 categories (A–K)

### Change
Add 4 items (B7–B10) worth 0.5 points each = +2 points.

| ID | Item | Full (0.5) | Half (0.25) | Zero |
|----|------|-----------|-------------|------|
| B7 | Async Hook usage | `async: true` on heavy hooks (e.g. Stop) | Mentioned but not applied | None |
| B8 | HTTP Hook configuration | Remote validation/policy server connected | URL configured but untested | None |
| B9 | MCP Tool Matching Hook | `mcp__*` pattern matching in PreToolUse/PostToolUse | Pattern exists but no logic | None |
| B10 | PreToolUse security gate | Blocks dangerous commands (`rm -rf /`, `DROP TABLE`, force push) | Partial pattern list | None |

### Grade Scale Adjustment
- Total: 46 → 48 points
- A: 44–48 (was 43–46)
- B: 35–43 (was 34–42)
- C: 26–34 (was 25–33)
- D: 16–25 (was 15–24)
- F: 0–15 (was 0–14)

### Rationale
- Async hooks: Prevent Stop hook from blocking Claude response when running heavy analysis
- HTTP hooks: Enable centralized team policy enforcement (e.g., lint rules, banned patterns)
- MCP matching: Intercept MCP tool calls for logging, validation, or augmentation
- PreToolUse security: Only blocking hook type — ideal for preventing destructive operations

---

## B. setup-templates.md — New Hook Templates

### Location
Insert after line 231 (after existing Stop hook template, before "### How the Auto-Handoff System Works").

### Templates to Add

#### B7: Async Hook Example
```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "node .claude/scripts/quality-report.js",
            "async": true,
            "timeout": 60
          }
        ]
      }
    ]
  }
}
```

Usage: Background quality analysis on session end. Results written to `.claude/reports/`.

#### B8: HTTP Hook Example
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash|Write",
        "hooks": [
          {
            "type": "http",
            "url": "https://policy.internal/validate",
            "method": "POST",
            "headers": { "Authorization": "Bearer {{POLICY_TOKEN}}" },
            "timeout": 5
          }
        ]
      }
    ]
  }
}
```

Usage: Remote policy server validates tool calls before execution.

#### B9: MCP Tool Matching Hook Example
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "mcp__plugin_serena_serena__replace_symbol_body|mcp__plugin_serena_serena__replace_content",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"[MCP Edit Logged] $(date +%H:%M:%S)\" >> .claude/logs/mcp-edits.log"
          }
        ]
      }
    ]
  }
}
```

Usage: Audit trail for MCP-driven code modifications.

#### B10: PreToolUse Security Gate Example
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo $CLAUDE_TOOL_INPUT | node -e \"const s=require('fs').readFileSync('/dev/stdin','utf8'); const blocked = ['rm -rf /','DROP TABLE','--force','--no-verify','chmod 777']; const cmd = JSON.parse(s).command||''; if(blocked.some(b=>cmd.includes(b))){console.error('BLOCKED: dangerous command detected'); process.exit(2)}\""
          }
        ]
      }
    ]
  }
}
```

Usage: Blocks destructive bash commands. Exit code 2 = block the tool call.

---

## C. code-polish — Self-Improving Pattern

### Current State
5-step workflow: identify → snapshot → clean → validate → report.

### Change
Add **Step 6: Pattern Learning** after the report step.

#### Step 6: 패턴 학습

After completing the polish report:

1. Read `.claude/polish-patterns.json` (create if missing)
2. For each change made, record:
   ```json
   {
     "patterns": [
       {
         "category": "dead-code|quality|performance|type-safety",
         "pattern": "unused-import",
         "file": "src/foo.ts",
         "count": 1,
         "lastSeen": "2026-03-16"
       }
     ]
   }
   ```
3. Increment `count` for patterns already recorded
4. On next run, read this file first and **prioritize high-frequency patterns**
5. If any pattern has `count >= 3`, add to report:
   ```
   ### 반복 패턴 경고
   - `unused-import` — 3회 이상 반복. 팀 컨벤션 또는 lint 규칙 추가 권장.
   ```

#### Modification to Step 1
Add before file scanning:
```
# 패턴 학습 데이터 로드 (있으면)
if [ -f .claude/polish-patterns.json ]; then
  # 고빈도 패턴 우선 검사 순서 결정
fi
```

#### New Rule
Add Rule 11:
> **패턴 학습** — `.claude/polish-patterns.json`에 발견 패턴을 기록한다. 3회 이상 반복되면 리포트에 경고하고 lint 규칙 추가를 권장한다.

---

## D. Progressive Disclosure Optimization

### Current State
Most SKILL.md descriptions are long, containing trigger keywords inline.

### Change
Separate concerns in frontmatter: `description` for Claude's matching decision (short, specific), triggers remain but description focuses on the skill's purpose.

#### Before (code-polish example):
```yaml
description: Automatically reviews and polishes changed code — cleanup dead code, optimize performance, refactor for readability. Auto-triggers via Stop hook after coding sessions. Triggers on "코드정리", "코드 최적화", "리팩토링", "code polish", "cleanup code", "정리해", "optimize code", "코드 다듬어", "polish".
```

#### After:
```yaml
description: Auto-cleanup changed source files — remove dead code, improve naming, optimize loops, strengthen types. Validates with lint/test, rolls back on failure. Auto-triggers via Stop hook.
```

Trigger keywords remain in the description text but are more naturally integrated. The key change is making the description focus on WHAT the skill does rather than listing trigger words.

#### All Skills — Updated Descriptions

| Skill | New Description |
|-------|----------------|
| code-polish | Auto-cleanup changed source files — remove dead code, improve naming, optimize loops, strengthen types. Validates with lint/test, rolls back on failure. Auto-triggers via Stop hook. |
| dev-start | Session initializer — loads handoff state, shows git/progress status, suggests next work, detects complex tasks for Agent Team. |
| dev-wrap | Session wrap-up — commits changes, shuts down Agent Teams, updates progress/features/changelog, generates handoff for next session. |
| project-setup | Scores project setup (48-point checklist), recommends improvements by tech stack, generates missing configs with approval. |
| skill-register | Discovers skills, registers them to CLAUDE.md trigger table, creates symlinks, verifies resolution. |
| team-setup | Analyzes project structure, identifies domains, generates tailored Agent Team skills with hooks and rules. |
| team-dispatcher | Analyzes user request to select the right Agent Team (bugfix/review/refactor/performance/feature), dispatches to correct team skill. |
| usage-guide | Generates project-specific unified usage guide covering all skills, MCP plugins, hooks, agents, and commands. |

---

## E. New Skill: skill-eval/SKILL.md

### Purpose
Run evals, benchmarks, and A/B comparisons on skills to measure quality and track performance across model updates.

### Workflow

#### Mode 1: Eval (Test Cases)
1. User specifies target skill and test scenarios
2. For each scenario, spawn independent agent with clean context
3. Agent invokes the skill with the scenario input
4. Measure: did it follow the skill instructions? Did it produce correct output?
5. Report pass/fail per scenario

#### Mode 2: Benchmark
1. Run all eval scenarios for a skill
2. Track: pass rate, token usage, elapsed time
3. Store results in `.claude/evals/{skill-name}/benchmark-{date}.json`
4. Compare against previous benchmarks if available

#### Mode 3: Compare (A/B)
1. Run same scenarios against two versions of a skill
2. Side-by-side comparison of pass rate, tokens, time
3. Recommend which version is better

### Triggers
"스킬 평가", "eval skill", "benchmark skill", "스킬 테스트", "skill eval", "스킬 벤치마크"

---

## F. New Skill: security-guard/SKILL.md

### Purpose
Automated security prevention — scan code changes for OWASP Top 10 vulnerabilities and block dangerous operations.

### Components

#### Component 1: PreToolUse Hook Template
Block dangerous bash commands (provided in setup-templates as B10).

#### Component 2: Code Security Scan (On-Demand + Auto)
After code changes, scan for:
- SQL injection (string concatenation in queries)
- XSS (unescaped user input in templates)
- Command injection (user input in shell commands)
- Hardcoded secrets (API keys, passwords, tokens)
- Insecure dependencies (known CVE patterns)
- Path traversal (user input in file paths)
- SSRF (user-controlled URLs in requests)

#### Workflow
1. Identify changed files (same method as code-polish Step 1)
2. For each file, run security pattern checks
3. Classify findings: CRITICAL / WARNING / INFO
4. For CRITICAL: suggest immediate fix with code diff
5. For WARNING: document and recommend fix
6. Report in structured format

#### Integration
- Can auto-trigger via Stop hook (after code-polish, before dev-wrap)
- Or manual invocation via triggers

### Triggers
"보안 검사", "security check", "security scan", "보안 스캔", "security guard", "취약점 검사"

---

## Implementation Order

1. **A + B**: checklist.md + setup-templates.md (coupled, do together)
2. **D**: Progressive Disclosure (quick frontmatter edits across all skills)
3. **C**: code-polish self-improving (add Step 6 + Rule 11)
4. **F**: security-guard new skill
5. **E**: skill-eval new skill

---

## Files Changed

| File | Change Type |
|------|-------------|
| project-setup/checklist.md | Edit — add B7-B10 |
| project-setup/setup-templates.md | Edit — add 4 hook templates |
| code-polish/SKILL.md | Edit — add Step 6, Rule 11 |
| dev-start/SKILL.md | Edit — update frontmatter description |
| dev-wrap/SKILL.md | Edit — update frontmatter description |
| project-setup/SKILL.md | Edit — update frontmatter description + score reference |
| skill-register/SKILL.md | Edit — update frontmatter description |
| team-setup/SKILL.md | Edit — update frontmatter description |
| team-dispatcher/SKILL.md | Edit — update frontmatter description |
| usage-guide/SKILL.md | Edit — update frontmatter description |
| skill-eval/SKILL.md | New file |
| security-guard/SKILL.md | New file |
| README.md | Edit — add new skills to table |
