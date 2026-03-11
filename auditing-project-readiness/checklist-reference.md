# Audit Checklist Reference

Detailed scoring criteria for each diagnostic item.
Use this to make consistent pass/partial/fail judgments.

---

## Scoring Rules

- **✅ Present**: Item fully satisfies criteria
- **⚠️ Partial**: Item exists but incomplete, outdated, or misconfigured
- **❌ Missing**: Item does not exist or is non-functional

---

## A. Claude Code Infrastructure

### A1: CLAUDE.md exists and is < 200 lines
- ✅ File exists, under 200 lines, contains project-specific info
- ⚠️ File exists but > 200 lines (too large for efficient context use)
- ⚠️ File exists but contains only generic boilerplate (not project-specific)
- ❌ No CLAUDE.md at project root

### A2: CLAUDE.md contains build/test commands
- ✅ Commands section with runnable commands (dev, build, test, lint)
- ⚠️ Some commands listed but incomplete (missing test or lint)
- ❌ No commands section

### A3: CLAUDE.md contains architecture overview
- ✅ Tech stack, directory structure, key patterns documented
- ⚠️ Only tech stack listed, no directory or pattern info
- ❌ No architecture info

### A4: CLAUDE.md contains coding conventions
- ✅ Specific rules (naming, imports, state management patterns)
- ⚠️ Generic rules only ("write clean code")
- ❌ No conventions

### A5: CLAUDE.md contains session start/end protocol
- ✅ Explicit steps for starting and ending sessions (read progress, update docs)
- ⚠️ Partial (only start or only end documented)
- ❌ No session protocol

### A6: CLAUDE.md contains context management rules
- ✅ Thresholds documented (when to /compact, /clear) + files to avoid reading
- ⚠️ General mention without specific thresholds
- ❌ No context management info

### A7: CLAUDE.md contains skill trigger table
- ✅ Table mapping keywords to skills with actions
- ⚠️ Skills mentioned but no trigger mapping
- ❌ No skill triggers

### A8: .claude/settings.json exists with hooks
- ✅ File exists with at least one hook configured
- ⚠️ File exists but no hooks (only permissions or env)
- ❌ File does not exist

### A9: SessionStart hook configured
- ✅ Hook runs on session start (shows git status, progress, pending items)
- ⚠️ Hook exists but shows minimal info
- ❌ No SessionStart hook

### A10: Stop hook configured
- ✅ Hook reminds to update progress/specs/changelog before ending
- ⚠️ Hook exists but doesn't check documentation updates
- ❌ No Stop hook

### A11: settings.local.json is clean
- ✅ Permissions use wildcard patterns (< 50 entries)
- ⚠️ 50-100 entries, some one-off commands mixed with patterns
- ❌ > 100 entries or full of one-off specific commands (PID kills, specific file paths)

---

## B. Skills & Agents

### B1: Planning skill exists
- ✅ Skill that enforces "plan before code" with explicit "do not write code" rule
- ⚠️ Skill exists but doesn't prevent premature coding
- ❌ No planning skill

### B2: Progress/handoff skill exists
- ✅ Skill for recording session progress (updates progress.txt, features-list)
- ⚠️ Skill exists but doesn't update tracking files
- ❌ No progress tracking skill

### B3: Context management skill exists
- ✅ Skill with thresholds, stuck-detection, and recovery protocol
- ⚠️ Skill mentions context but no specific thresholds or recovery steps
- ❌ No context management skill

### B4: Skills match project tech stack
- ✅ All skills reference correct framework/language (e.g., Next.js commands for a Next.js project)
- ⚠️ Some skills are generic or partially wrong framework
- ❌ Skills reference wrong framework (e.g., Flutter commands in a React project)

### B5: Code review agent defined
- ✅ Agent in .claude/agents/ with project-specific conventions checklist
- ⚠️ Agent exists but generic (no project conventions)
- ❌ No code review agent

### B6: Bug analysis agent defined
- ✅ Agent with structured hypothesis approach and project-specific bug patterns
- ⚠️ Agent exists but generic
- ❌ No bug analysis agent

### B7: Agents reference project-specific conventions
- ✅ Agents mention specific project rules (import paths, state management, ORM boundaries)
- ⚠️ Agents have some project context but miss key conventions
- ❌ Agents are completely generic

### B8: Slash commands exist
- ✅ At least 2 commands in .claude/commands/ for common workflows
- ⚠️ 1 command exists
- ❌ No commands directory or empty

---

## C. Spec Documents

### C1: PRD or requirements document
- ✅ Document listing all features with current status (complete/in-progress/planned)
- ⚠️ Requirements exist but no status tracking
- ❌ No requirements document

### C2: Feature specs exist
- ✅ Per-feature spec files with: description, flow, related code, completion criteria
- ⚠️ Some feature docs exist but missing key sections (no completion criteria, no code refs)
- ❌ No per-feature specs

### C3: API specs exist
- ✅ Endpoint documentation with: method, path, request/response shapes, auth, errors
- ⚠️ API docs exist but incomplete (missing request/response shapes)
- ❌ No API documentation

### C4: DB schema docs exist
- ✅ Table/column documentation matching actual schema
- ⚠️ Schema docs exist but outdated (don't match current schema)
- ❌ No schema documentation

### C5: Specs include completion criteria
- ✅ Feature specs have testable acceptance criteria
- ⚠️ Some criteria exist but are vague ("works correctly")
- ❌ No completion criteria in any spec

### C6: Specs include code file references
- ✅ Feature specs list relevant source files (paths to components, APIs, services)
- ⚠️ Some file references but incomplete
- ❌ No code references in specs

### C7: Specs are < 500 lines each
- ✅ All spec files under 500 lines (context-friendly)
- ⚠️ Some specs are 500-1000 lines
- ❌ Spec files > 1000 lines (context-heavy, defeats purpose)

---

## D. Operational Documents

### D1: Progress tracking
- ✅ File with structured session log (date, what done, what next, issues, commits)
- ⚠️ Some progress notes but unstructured or in random locations
- ❌ No progress tracking file

### D2: Feature/bug tracking
- ✅ File with items categorized by status (FAILING/IN_PROGRESS/PASSING/BLOCKED)
- ⚠️ TODO list exists but no status categorization
- ❌ No tracking file

### D3: Changelog
- ✅ CHANGELOG.md with dated entries tracking what changed and why
- ⚠️ Changelog exists but sparse or outdated
- ❌ No changelog

### D4: Architecture Decision Records
- ✅ ADR files documenting key technical decisions with context and consequences
- ⚠️ Some decisions mentioned in docs but not structured as ADRs
- ❌ No ADRs

### D5: Agent team briefing document
- ✅ Compact doc (< 200 lines) with: project summary, tech stack, hard rules, key dirs, current state refs
- ⚠️ Briefing info scattered across CLAUDE.md and README (not compact)
- ❌ No briefing document (teammates would need to read full README)

---

## E. Context Efficiency

### E1: No oversized required docs
- ✅ No file > 50KB that agents routinely need to read
- ⚠️ 1-2 files > 50KB (should be split into feature specs)
- ❌ Multiple large files that agents must read for context

### E2: Multiple context sources
- ✅ Feature specs, CLAUDE.md, and briefing doc provide focused context
- ⚠️ CLAUDE.md + README are the only context sources
- ❌ README.md is the sole source of project context

### E3: Feature-level docs exist
- ✅ Per-feature docs instead of monolithic architecture documents
- ⚠️ Mix of monolithic and feature-level docs
- ❌ Only monolithic docs (one file for entire system)

### E4: Large file warnings in CLAUDE.md
- ✅ CLAUDE.md explicitly lists files agents should NOT read in full
- ⚠️ General advice about context but no specific file warnings
- ❌ No warnings about large files

### E5: Compact/clear strategy documented
- ✅ Specific thresholds and steps for /compact and /clear usage
- ⚠️ Mentioned but no specific thresholds
- ❌ No compact/clear strategy

---

## F. Testing & Quality

### F1: Test framework configured
- ✅ Test config file exists (jest.config, vitest.config, etc.) with project-specific settings
- ⚠️ Default config with no customization
- ❌ No test framework configured

### F2: Test files exist
- ✅ > 10 test files with actual test cases
- ⚠️ 1-10 test files
- ❌ No test files or only empty test files

### F3: E2E test setup
- ✅ E2E framework configured (Playwright, Cypress) with actual test scenarios
- ⚠️ Framework configured but few/no actual tests
- ❌ No E2E framework

### F4: Test commands in CLAUDE.md
- ✅ Test commands documented with single-file test pattern
- ⚠️ Only basic test command, no single-file pattern
- ❌ Test commands not documented

### F5: CI/CD pipeline
- ✅ GitHub Actions or equivalent with lint + test + build steps
- ⚠️ Pipeline exists but limited (build only, no tests)
- ❌ No CI/CD pipeline
