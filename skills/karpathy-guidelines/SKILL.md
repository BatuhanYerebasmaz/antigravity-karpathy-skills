---
name: karpathy-guidelines
description: "Behavioral guidelines to reduce common LLM coding mistakes based on Andrej Karpathy's principles. Use when writing, reviewing, refactoring, or planning code changes to avoid overengineering, make surgical edits, surface hidden assumptions, and define verifiable success criteria."
---

# Karpathy Guidelines for Antigravity

Behavioral guidelines to reduce common LLM coding mistakes, derived from [Andrej Karpathy's observations](https://x.com/karpathy/status/2015883857489522876) on LLM coding pitfalls.

**Tradeoff:** These guidelines bias toward caution, precision, and simplicity over speed or premature flexibility. For trivial tasks, apply sensible engineering judgment.

---

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before writing or editing code:
- **Surface assumptions explicitly**: If the prompt leaves scope, data models, or behavior unspecified, state your assumptions or ask for clarification.
- **Present alternatives & ambiguity**: If multiple valid interpretations exist, outline the choices—do not silently pick one and implement it.
- **Push back constructively**: If a simpler, less error-prone path exists than what was initially asked, suggest it clearly.
- **Stop when confused**: If requirements contradict existing code or intentions are ambiguous, pause immediately and ask. In Antigravity, use `ask_question` or direct text to clarify before modifying files.

---

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

Combat the natural LLM tendency toward overengineering and abstraction bloat:
- **Zero speculative features**: Never add features, options, or hooks that were not explicitly requested.
- **No single-use abstractions**: Do not introduce classes, interfaces, strategy patterns, or helper indirection for code that is used in exactly one place.
- **No speculative configurability**: Avoid adding config flags, parameters, or environment toggles "just in case".
- **No paranoia error handling**: Avoid handling impossible scenarios that add dozens of lines of boilerplate.
- **Senior engineer test**: If an implementation took 200 lines and could cleanly be done in 50 lines, rewrite it to 50.

> *Rule of thumb: Write code that solves today's problem cleanly. Refactor when tomorrow's requirements actually arrive.*

---

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When modifying existing code:
- **No drive-by refactoring**: Do not reformat adjacent lines, "clean up" unrelated comments, or modernize styles outside the exact target.
- **Match existing conventions**: Adhere strictly to the existing file's style (quote style, indentation, naming conventions, typing level), even if you would personally write it differently.
- **Preserve working code**: Do not alter or refactor working code unless the user explicitly requested it.
- **Surface unrelated issues without touching**: If you discover unrelated bugs or dead code, mention them in your response—do not silently alter or delete them.
- **Clean up your own orphans**: If your modification makes an import, variable, or function obsolete, remove it. Do not touch pre-existing dead code.

> *Diff test: Every single changed line in your diff must trace directly to the user's explicit request.*

---

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform vague or imperative tasks into verifiable, testable milestones before making changes:
- Instead of *"Fix the bug"*, define: *"Write a reproduction test/case → verify it fails → apply the fix → verify the test passes."*
- Instead of *"Add feature"*, define: *"Define expected input/output → implement → verify with tests/linters."*
- Instead of *"Refactor X"*, define: *"Verify existing test suite passes before and after the change."*

For multi-step implementations, maintain a lightweight verification loop:
```text
1. [Step 1] → verify: [exact command or test check]
2. [Step 2] → verify: [exact command or test check]
3. [Step 3] → verify: [exact command or test check]
```

Use tools (such as test runners, linters, or typecheckers via `run_command`) to autonomously verify each step before reporting completion.

---

## Antigravity Native Integration

| Karpathy Principle | Antigravity Native Practice |
| :--- | :--- |
| **Think Before Coding** | Use `ask_question` when choices are ambiguous; read files with `read_file` to inspect ground truth before assuming. |
| **Simplicity First** | Keep diffs small; prefer straightforward functions over complex OOP hierarchies; avoid unnecessary dependencies. |
| **Surgical Changes** | Use `edit_file` with precise string targets; do not modify unrelated functions, comments, or imports. |
| **Goal-Driven Execution** | Run linters/tests via `run_command` (e.g. `selene`, `stylua`, `pytest`, `npm test`) to confirm fixes before finishing. |

---

## Detailed Examples & Anti-Patterns

For complete code walkthroughs comparing common LLM mistakes against the surgical Karpathy approach across all 4 principles, see:
- [references/examples.md](references/examples.md)
