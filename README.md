# DZt-eXpression-Language

> **DevZoft Expression Language** — declare AI workflows, compile them into structured prompts, and execute them through your installed AI agents.

**DZx** is a small, declarative language for building AI instructions and OpenCode workflows. You write a `.dzx` file, DZx parses it into an AST, compiles it into a structured prompt, and runs it against your chosen agent (OpenCode or GitHub Copilot CLI). No glue code, no prompt spaghetti.

![Version](https://img.shields.io/npm/v/%40devzoft/dzx?style=flat-square&logo=npm&color=2dd4bf)
![Node](https://img.shields.io/node/v/%40devzoft/dzx?style=flat-square&logo=node.js&color=3c873a)
![License](https://img.shields.io/npm/l/%40devzoft/dzx?style=flat-square&color=blue)
![Platform](https://img.shields.io/badge/platform-win32%20%7C%20linux%20%7C%20macos-lightgrey?style=flat-square)
![Zero deps](https://img.shields.io/badge/runtime_dependencies-0-22d3ee?style=flat-square)

---

## Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [CLI Reference](#cli-reference)
- [Agent Backends](#agent-backends)
- [Configuration](#configuration)
- [The DZx Language](#the-dzx-language)
- [Workflow](#workflow)
- [Examples](#examples)
- [Development](#development)
- [Architecture](#architecture)
- [License](#license)

---

## Features

- **Declarative by design** — describe *what* the agent should do, not how.
- **Deterministic compilation** — `.dzx` → AST → structured prompt, every time.
- **Two AI backends** — execute via **OpenCode** (`opencode run`) or **GitHub Copilot CLI** (`copilot -p`).
- **Built-in SEO auditor** — crawl any public URL and get an instant on-page SEO report.
- **Local validation** — `check` and `compile` run fully offline; no agent required.
- **Zero runtime dependencies** — lean install, fast startup.
- **Cross-platform** — native `.exe` detection and `.cmd` shim fallback on Windows, direct spawn on POSIX.
- **Precise diagnostics** — parser errors report exact `line:column` locations.

---

## Installation

**Requirements:** Node.js **20 or newer**.

Install globally:

```bash
npm i -g @devzoft/dzx
```

That's it — no build step, no compilation, ready to use.

### Upgrading

```bash
npm i -g @devzoft/dzx@latest
```
---

## Quick Start

```bash
# 1. Detect your installed AI agents
dzx agents

# 2. Register the agent you want to use
dzx agents select opencode        # or: copilot

# 3. Validate your workflow file
dzx check examples/developer.dzx

# 4. Preview the generated prompt
dzx compile examples/developer.dzx

# 5. Execute the workflow through your agent
dzx run examples/developer.dzx
```

---

## CLI Reference

```
DZx 0.1.0
Declarative workflows for OpenCode

Usage:
  dzx check <file.dzx>
  dzx compile <file.dzx>
  dzx run <file.dzx>
  dzx seo <url>
  dzx agents
  dzx agents select <copilot|opencode>
  dzx version
```

| Command | Argument | Description | Requires Agent |
| --- | --- | --- | --- |
| `dzx` / `dzx help` / `-h` / `--help` | — | Print help and usage | — |
| `dzx version` | — | Print the installed DZx version | — |
| `dzx check <file.dzx>` | file path | Parse and validate a workflow file | No |
| `dzx compile <file.dzx>` | file path | Parse + compile into a structured prompt and print it | No |
| `dzx run <file.dzx>` | file path | Parse + compile + execute the prompt through the selected agent | Yes |
| `dzx seo <url>` | URL | Fetch a public page and run an on-page SEO audit | No |
| `dzx agents` | — | Detect installed agents and show the selection table | No |
| `dzx agents select <agent>` | `copilot` \| `opencode` | Register the agent used by `dzx run` | No |

### Command details

#### `dzx check <file.dzx>`

Validates a workflow file locally — lexing and parsing with full `line:column` diagnostics.

```
OK: examples/developer.dzx
```

Exit code `0` on success, `1` on parse/validation errors.

#### `dzx compile <file.dzx>`

Parses and compiles the file to a prompt, printing the exact text that would be sent to the agent. Great for debugging and prompt inspection.

#### `dzx run <file.dzx>`

Runs the compiled prompt through the selected agent (see [Agent Backends](#agent-backends)). The process exits with the agent's own exit code, so it composes cleanly with CI pipelines.

#### `dzx seo <url>`

Runs an on-page SEO audit against any public `http(s)` URL:

```
DZx SEO Audit Report
====================
URL: https://example.com
Final URL: https://example.com/
HTTP status: 200
Title: ...

Summary: 5 passed, 4 warnings, 2 failures

[PASS] HTTP status: 200 OK
[PASS] Title: 42 characters
```

Checks include: HTTP status, title length (30–60), meta description (120–160), canonical URL, HTML language, viewport, exactly one H1, image `alt` text, Open Graph title/description, and robots directives.

#### `dzx agents`

Detects OpenCode and GitHub Copilot CLI installations and prints a selection table — the `*` marks your currently selected agent:

```
DZx agents
==========
* opencode: OpenCode - installed (1.18.32)
  copilot: GitHub Copilot CLI - not installed
```

#### `dzx agents select <copilot|opencode>`

Registers the agent used by `dzx run`. Validates the agent is installed before saving. Persisted to [Configuration](#configuration).

---

## Agent Backends

`dzx run` dispatches the compiled prompt to exactly one backend:

| Backend | Command | Notes |
| --- | --- | --- |
| **OpenCode** | `opencode run <prompt>` | Recommended; used non-interactively |
| **GitHub Copilot CLI** | `copilot -p <prompt>` | Requires GitHub Copilot CLI with a session |

On Windows, DZx prefers the native `.exe` and falls back to `.cmd` shims through the shell to avoid prompt mangling. On POSIX, agents are spawned directly.

> **Note:** `check` and `compile` work offline and never touch an agent. Only `run` needs a registered, installed backend.

---

## Configuration

DZx stores its agent selection in a single JSON file:

```
~/.dzx/agents.json
```

```json
{
  "selected": "opencode",
  "registered": ["opencode"]
}
```

The directory is created automatically on first `agents select`. If the file is missing or corrupt, DZx treats it as an empty registration and guides you to run `dzx agents select opencode`.

---

## The DZx Language

### Syntax at a glance

| Construct | Example |
| --- | --- |
| Command | `@name "Label" { ... }` |
| Object / config block | `{ key: value }` |
| List | `[ a b c ]` |
| Params (positional + named) | `(fast, provider: "openai")` |
| Key assignment | `:` or `=` |
| Strings | `'single'` / `"double"` |
| Comments | `//` and `#` |
| Separators | commas/semicolons optional where unambiguous |

### Example

```dzx
@agent "Developer" {
  goal: "Build a reliable application";

  project: {
    name: "DZx Demo"
    framework: "Next.js"
    language: "TypeScript"
  }

  tools: [filesystem terminal git]
  model: (fast, provider: "openai")
}
```

### Rules

- Keys must be **unique** within an object or parameter list.
- Values are strings, numbers, booleans, `null`, lists, objects, or params.
- String escapes: `\n`, `\r`, `\t`, `\\`, `\"`, `\'`.
- Unterminated blocks, invalid values, duplicate keys, and unexpected characters produce errors with exact `line:column` locations.

---

## Workflow

1. **Write** a `.dzx` file describing the agent and its brief.
2. **`dzx check`** — catch syntax errors offline.
3. **`dzx compile`** — inspect the exact prompt that will be sent.
4. **`dzx run`** — hand the prompt to OpenCode or Copilot and get the result.

---

## Examples

The package ships a realistic end-to-end example:

```bash
dzx check examples/developer.dzx
dzx compile examples/developer.dzx
dzx run examples/developer.dzx
```

```dzx
// Define the developer agent and its project brief.
@agent "Developer" {
  goal: "Build a production-ready Next.js application"

  project: {
    name: "DZx Demo"
    framework: "Next.js"
    language: "TypeScript"
    database: "SQLite"
  }

  // Tools the agent may use while working.
  tools: [
    filesystem
    terminal
    git
  ]

  // Required delivery workflow.
  workflow: [
    analyze
    plan
    implement
    test
    fix
  ]

  // Quality and product requirements.
  constraints: [
    "responsive"
    "type-safe"
    "production-ready"
  ]
}
```

---

## Architecture

```
.dzx ──▶ Lexer ──▶ Parser ──▶ AST ──▶ Compiler ──▶ Agent (OpenCode / Copilot)
```

```
src/
├── cli.ts         CLI entry point: argument parsing, help, commands
├── tokenizer.ts   Lexer: tokens, strings, numbers, comments
├── parser.ts      Recursive-descent parser → AST
├── compiler.ts    AST → structured prompt text
├── runner.ts      Spawns OpenCode / Copilot with the compiled prompt
├── seo.ts         On-page SEO auditor + report formatter
└── agents.ts      Agent detection, Windows path resolution, ~/.dzx config
```

DZx v1.0 intentionally uses **OpenCode** as its primary AI execution backend, with GitHub Copilot CLI as an alternative — one language, multiple agents.

---

## License

[MIT](./LICENSE) · Copyright © Amal K P · Built by [DevZoft](https://instagram.com/devzoft)

- Developer: [Amal K P](https://instagram.com/espoir.__._)
- Organization: [DevZoft](https://instagram.com/devzoft)
