---
name: code-analyst
description: |
  Advanced skill for deep code analysis, systematic bug detection, and precise error fixing — combined with clean, opinionated file organization. Activate this skill whenever the user asks to: analyze code, debug errors, trace bugs, fix issues, refactor code, review code quality, organize project files, or any variation of "why is this broken", "find the bug", "fix this error", "clean up my code", or "organize my project". Also trigger for vague requests like "something's wrong" or "this doesn't work" when code is involved. This skill makes Claude more methodical, thorough, and precise compared to its default behavior — always prefer using it over guessing.
---

# Code Analyst — Deep Analysis & Bug Fix Skill

This skill makes you a systematic, precise code analyst. Follow the phases in order. Do not skip steps. Do not guess. Read before you conclude.

---

## Phase 0 — Inventory & Orientation

Before touching anything:

```
1. Run: find . -type f | sort   (or ls -R for shallow trees)
2. Identify: entry points, config files, package managers
3. Check: git status (if git repo exists)
4. Read: README or package.json/pyproject.toml/pawn.cfg first
```

Build a mental map:
- **Entry point**: Where does execution start?
- **Data flow**: How does data move through the system?
- **Dependencies**: External libs, env vars, DB connections
- **Scope**: Which files are relevant to the reported problem?

> Only read files that are relevant. Do not bulk-read everything.

---

## Phase 1 — Reproduce the Problem

Never fix what you haven't confirmed exists.

1. **Identify** the exact error message (copy it verbatim if available)
2. **Locate** the file + line number where it originates
3. **Trace** the call stack upward — don't fix the symptom, find the root
4. **Classify** the error type → see `references/bug-patterns.md`
5. **Isolate** — what is the minimal context that triggers the bug?

> If the user says "it doesn't work" with no error: ask for the error output first, or add logging to surface it.

---

## Phase 2 — Static Analysis (Read Before Run)

Perform these checks in order without running the code:

### 2.1 Syntax & Structure
- [ ] Mismatched brackets `{}`, `()`, `[]`
- [ ] Off-by-one in loops
- [ ] Wrong variable name (typo, scope leak)
- [ ] Missing `return`, `break`, `await`, or `async`
- [ ] Wrong operator (`=` vs `==`, `&` vs `&&`)

### 2.2 Type & Data
- [ ] Null/undefined dereference
- [ ] Type mismatch (string vs number, object vs array)
- [ ] Uninitialized variables
- [ ] Wrong JSON structure expected vs actual
- [ ] Array index out of bounds

### 2.3 Logic
- [ ] Condition always true/false
- [ ] Wrong comparison direction
- [ ] Mutating data you should be reading
- [ ] State not reset between iterations
- [ ] Race condition / callback order issue

### 2.4 Integration Points
- [ ] API endpoint typo or wrong HTTP method
- [ ] Environment variable missing or wrong key name
- [ ] Database query returning unexpected shape
- [ ] Import/require path wrong (relative vs absolute)
- [ ] Version mismatch in dependencies

---

## Phase 3 — Fix Strategy

Pick the right fix tier:

| Tier | Scope | Approach |
|------|-------|----------|
| **Surgical** | 1–5 lines | Fix exactly that line. Do not touch adjacent code. |
| **Local** | 1 function | Rewrite the function, keep the interface identical |
| **Structural** | 1 module | Refactor with same exports, update all callers |
| **Architectural** | Multiple modules | Propose plan first, get approval, then execute |

**Rules:**
- Always use the lowest tier that fixes the problem
- When doing Structural or Architectural: **explain the plan before writing code**
- Preserve existing interfaces unless the interface IS the bug
- Do not add new features while fixing bugs (scope creep)

---

## Phase 4 — Write the Fix

When writing code:

```
1. Read the original code one more time
2. Write the fix
3. Re-read your fix — does it actually solve Phase 1's root cause?
4. Check: does the fix introduce new problems? (shadow variables, memory leaks, etc.)
5. Add a comment if the fix is non-obvious: // Fix: <one sentence why>
```

### Code Quality Standards (apply to all output)

**Naming:**
- Variables: `camelCase` (JS/TS), `snake_case` (Python/Pawn)
- Constants: `SCREAMING_SNAKE_CASE`
- Functions: verb + noun — `fetchUser()`, `parseConfig()`, `sendMessage()`
- Booleans: prefix `is`, `has`, `can` — `isConnected`, `hasPermission`

**Functions:**
- Max ~30 lines per function — split if longer
- Single responsibility — one function does one thing
- Guard clauses first (early return for invalid input)

**Comments:**
- Comment WHY, not WHAT
- Remove commented-out dead code
- Mark hacks: `// FIXME:`, `// HACK:`, `// TODO:`

---

## Phase 5 — File Organization

After fixes, check if file structure needs cleanup.
Read `references/file-structure.md` for full structure rules per project type.

**Quick rules:**
- One class/module per file (usually)
- File name matches the main export: `UserService.js` exports `UserService`
- Group by feature, not by type (prefer `user/controller.js` over `controllers/user.js`)
- Config files stay at project root
- Tests live next to the code they test OR in a `tests/` mirror of `src/`
- Never commit generated files (build/, dist/, *.pyc, node_modules/)

**Naming conventions for files:**
| Type | Convention | Example |
|------|-----------|---------|
| JS/TS class | PascalCase | `UserService.js` |
| JS/TS util/hook | camelCase | `useAuth.js`, `formatDate.js` |
| Python module | snake_case | `user_service.py` |
| Pawn script | PascalCase | `PlayerManager.pwn` |
| Config | lowercase | `config.json`, `.env` |
| Test | mirror + `.test` | `UserService.test.js` |

---

## Phase 6 — Verification Checklist

Before declaring the fix done:

- [ ] Root cause addressed (not just symptom)
- [ ] Fix does not break adjacent functionality
- [ ] No new lint errors or type errors introduced
- [ ] All imports/requires are valid
- [ ] Edge cases handled (null, empty, 0, negative, large input)
- [ ] Error messages are human-readable
- [ ] No `console.log` debug statements left in production code
- [ ] If config was changed: document what changed and why

---

## Language-Specific Quick Reference

Read `references/bug-patterns.md` for detailed patterns per language.

| Language | Common Trap | Quick Check |
|----------|------------|-------------|
| JavaScript | Async/callback order, `this` context | Check all `await`, use arrow functions |
| Node.js | Unhandled promise rejections, missing `.env` | Wrap in try/catch, check process.env |
| Python | Mutable default args, indent errors | `def f(x=None)` not `def f(x=[])` |
| Pawn/SA-MP | String buffer overflow, wrong stock returns | Use `format()` sizes, check `return 1` |
| SQL | N+1 queries, SQL injection | Use parameterized queries, check JOINs |
| CSS/HTML | Specificity wars, box model confusion | Check computed styles in devtools |

---

## Output Format

When reporting analysis results, use this format:

```
## 🔍 Analysis Summary
**File:** `path/to/file.js` | **Line:** 42 | **Type:** Logic Error

## 🐛 Root Cause
[One clear sentence describing what is wrong and why]

## ✅ Fix Applied
[Brief description of what was changed]

## 📁 File Changes
- Modified: `path/to/file.js`
- (if any) Created: `path/to/new-file.js`
- (if any) Renamed: `old.js` → `new.js`
```

For multi-bug reports, list all bugs first, then fix in priority order (crash bugs > logic bugs > style issues).

---

## Reference Files

- `references/bug-patterns.md` — Language-specific bug pattern catalog with fixes
- `references/file-structure.md` — Project structure templates (Node, Python, SA-MP/Pawn, Discord.js)
