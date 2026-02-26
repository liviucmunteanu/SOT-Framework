# SOT Framework

**SOT** (SOPs, Orchestration, Tools) is a framework for building reliable AI-powered workflows. It separates what AI is good at (reasoning, decision-making) from what it's bad at (consistent execution) — so your automations actually work.

## The Problem

When AI handles every step directly, errors compound. 90% accuracy per step = 59% success over 5 steps. The more steps, the worse it gets.

## The Solution

Three layers, strict separation:

| Layer | What | Where |
|---|---|---|
| **SOPs** | Markdown instructions defining *what* to do | `00-SOPs/` |
| **Orchestration** | AI reads SOPs, makes decisions, calls tools | `AGENTS.md` |
| **Tools** | Deterministic scripts that do the actual work | `00-TOOLS/` |

The AI never executes work directly — it reads the instructions and calls the right script. Scripts are deterministic: same input, same output, every time.

## Quick Start

1. Copy `AGENTS.md` into your project root
2. Create the directories: `00-SOPs/`, `00-TOOLS/`, `00-KBs/`
3. If you use APIs, copy `.env.example` to `.env` and add your credentials
4. Write your first SOP — describe the task, inputs, which tool to call, expected outputs
5. Build the tool — a Python script that handles one step deterministically
6. Point your AI agent at `AGENTS.md` and let it orchestrate

## Directory Structure

```
project-root/
├── AGENTS.md              # Agent instructions (the framework itself)
├── docs/                  # Human docs (architecture, decisions, specs)
├── 00-SOPs/               # What to do (Markdown instructions)
│   ├── core/              # Shared across workflows
│   └── <workflow-name>/   # Workflow-specific
├── 00-TOOLS/              # How to do it (deterministic scripts)
│   ├── core/              # Shared tools
│   └── <workflow-name>/   # Workflow-specific
├── 00-KBs/                # Institutional memory (gotchas, patterns, vendor notes)
│   └── security/          # Security rules and compliance
├── .tmp/                  # Temp files (gitignored, disposable)
├── .env                   # API keys (gitignored)
└── .env.example           # Documents required env vars
```

Start flat for simple projects (`sops/`, `tools/`, `kbs/`). Graduate to the full structure when you add a second workflow.

## Key Concepts

**Self-annealing** — When something breaks, the agent fixes the tool, verifies the fix, updates the SOP, and records the insight. The system gets stronger after every failure.

**Knowledge base** (`00-KBs/`) — Cross-cutting knowledge that spans multiple workflows. Gotchas, patterns, security rules, vendor quirks. Prevents the same mistake from happening twice in different places.

**Deliverables vs intermediates** — Temp files go in `.tmp/`. Final outputs go where you can access them (Google Sheets, Slides, shared drives, etc.).

## How It Works With AI Agents

`AGENTS.md` is the instruction file your AI agent reads. It tells the agent:
- What its role is (orchestrator, not executor)
- How the three layers connect
- How to operate (check for existing tools first, self-anneal, keep SOPs updated)
- Where to put things

Point any AI coding agent (Claude, Gemini, Cursor, etc.) at a project containing `AGENTS.md` and it will follow the framework automatically.

## License

MIT
