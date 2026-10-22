# Karpathy Guidelines for Antigravity

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Antigravity](https://img.shields.io/badge/Antigravity-Skill-orange.svg)](https://antigravity.google)

A specialized **Google Antigravity** skill implementing **Andrej Karpathy's coding guidelines** to prevent common LLM coding pitfalls, promote surgical changes, eliminate speculative overengineering, and enforce verifiable execution loops.

Based on [Andrej Karpathy's observations](https://x.com/karpathy/status/2015883857489522876) and adapted from [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills) for the Google Antigravity agent architecture.

---

## The Four Principles

| Principle | Core Directive | Addresses |
| :--- | :--- | :--- |
| **1. Think Before Coding** | *Don't assume. Don't hide confusion. Surface tradeoffs.* | Wrong assumptions, unasked questions, hidden edge cases |
| **2. Simplicity First** | *Minimum code that solves the problem. Nothing speculative.* | Over-abstraction, single-use patterns, premature complexity |
| **3. Surgical Changes** | *Touch only what you must. Clean up only your own mess.* | Drive-by refactoring, style drift, editing unrelated code/comments |
| **4. Goal-Driven Execution** | *Define success criteria. Loop until verified.* | "Make it work" vagueness; unverified broken implementations |

---

## Antigravity Native Integration

Unlike generic prompt snippets, this repository packages the guidelines as a first-class Antigravity skill using progressive disclosure:

- **`SKILL.md`**: Core operational guidelines and Antigravity tool mappings (`ask_question`, `read_file`, `edit_file`, `run_command`).
- **`references/examples.md`**: Condensed real-world wrong vs. surgical implementations loaded on-demand by the agent when deeper guidance is required.

---

## Installation

### Option 1: Workspace Installation (Project-Specific)

Place the skill inside your project's `.agent/skills/` or `.agents/skills/` directory:

```bash
# From your project root:
mkdir -p .agent/skills/karpathy-guidelines
git clone https://github.com/BatuhanYerebasmaz/antigravity-karpathy-skills.git temp-skill
cp -r temp-skill/skills/karpathy-guidelines/* .agent/skills/karpathy-guidelines/
rm -rf temp-skill
```

### Option 2: Global Installation (Available Across All Projects)

Install globally so every Antigravity session on your machine has access to the skill:

**Windows (PowerShell):**
```powershell
$target = "$HOME\.gemini\config\skills\karpathy-guidelines"
New-Item -ItemType Directory -Path $target -Force
git clone https://github.com/BatuhanYerebasmaz/antigravity-karpathy-skills.git temp-skill
Copy-Item -Path "temp-skill\skills\karpathy-guidelines\*" -Destination $target -Recurse -Force
Remove-Item -Path "temp-skill" -Recurse -Force
```

**macOS / Linux:**
```bash
target="$HOME/.gemini/config/skills/karpathy-guidelines"
mkdir -p "$target"
git clone https://github.com/BatuhanYerebasmaz/antigravity-karpathy-skills.git temp-skill
cp -r temp-skill/skills/karpathy-guidelines/* "$target/"
rm -rf temp-skill
```

---

## Directory Structure

```text
antigravity-karpathy-skills/
├── skills/
│   └── karpathy-guidelines/
│       ├── SKILL.md              # Main skill definition & Antigravity mappings
│       └── references/
│           └── examples.md       # Concrete good vs bad code examples
├── LICENSE                       # MIT License
└── README.md                     # Documentation
```

---

## Acknowledgements & Credits

- Derived from [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills) by Forrest Chang & Multica.
- Inspired by [Andrej Karpathy's post](https://x.com/karpathy/status/2015883857489522876) on LLM coding pitfalls.

---

## License

This project is licensed under the [MIT License](LICENSE).

The upstream project [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills) declares the MIT License in its README, `plugin.json`, and `SKILL.md` frontmatter (it does not ship a separate `LICENSE` file).
