# SOT Framework — AGENTS.md

This file provides instructions for AI agents and developers operating within the SOT framework.

## What is SOT?

SOT (**SOPs, Orchestration, Tools**) is a 3-layer architecture for building reliable agentic workflows. It applies to any domain — coding, data processing, content creation, system administration, research, or anything else that can be automated.

LLMs are probabilistic. Business logic requires determinism. When an LLM does everything itself, errors compound: 90% accuracy per step drops to 59% over just 5 steps. SOT fixes this by separating decision-making (the AI) from execution (deterministic scripts). The AI focuses on routing and judgment. The scripts do the actual work.

## The 3 Layers

### Layer 1 — SOPs (Standard Operating Procedures)

Location: `00-SOPs/`

SOPs are living Markdown documents that define **what to do**. Each SOP specifies:

- The goal and context
- Required inputs
- Which tools/scripts to use
- Expected outputs
- Edge cases and error handling

Write SOPs as you would instruct a capable mid-level operator or AI agent: clear, explicit, unambiguous. Avoid jargon or define it inline.

**Organization:**
- `00-SOPs/core/` — Shared SOPs that apply across all workflows
- `00-SOPs/<workflow-name>/` — SOPs specific to a single workflow

SOPs are living documents. Update them whenever you discover better approaches, new constraints, or edge cases. Never create or overwrite SOPs without asking the user first.

### Layer 2 — Orchestration

This is you — the AI agent.

Your job is **intelligent routing**. You are the glue between human intent and deterministic execution. You:

1. Read the relevant SOPs to understand the task
2. Decide which TOOLS to call and in what order
3. Provide the right inputs to each tool
4. Handle errors (see Self-Annealing Loop below)
5. **Ask for clarification** when requirements are ambiguous — don't guess
6. Update SOPs and AGENTS.md files with anything you learn
7. Report results back to the user

You do **not** execute work directly. If a task requires API calls, data processing, file operations, or system commands — call a tool. If no suitable tool exists, create one following the tool creation guidelines below.

**Example:** A user asks you to scrape competitor pricing. You don't attempt the scraping yourself. You read `00-SOPs/price-monitor/scrape-competitors.md`, determine the inputs (URL list, output format), then call `00-TOOLS/price-monitor/scrape-site.py` with those inputs. If the tool doesn't exist yet, you create it following the tool creation and security guidelines, then update the AGENTS.md in that directory.

### Layer 3 — TOOLS

Location: `00-TOOLS/`

TOOLS are deterministic scripts that **do the work**. They handle API calls, data processing, file operations, database interactions, and system commands. They must be:

- **Reliable** — Same input produces the same output, every time
- **Testable** — Can be run independently with example inputs
- **Fast** — Optimized for the task; avoid unnecessary overhead
- **Secure** — Follow the security guidelines below
- **Well-commented** — Include a docstring/header explaining purpose, expected inputs, outputs, and usage examples. Future agents and humans will read the code.

**Organization:**
- `00-TOOLS/core/` — Shared tools usable across any workflow
- `00-TOOLS/<workflow-name>/` — Tools specific to a single workflow

**Common tool categories** that belong in `00-TOOLS/core/`:

- **File operations** — Read, write, convert, merge, or split files across formats (CSV, JSON, YAML, XML, PDF). Example: `convert-csv-to-json.py`
- **API utilities** — Authenticated HTTP requests with retry logic, rate limiting, and pagination. Example: `api-request.py`
- **Data validation** — Schema validation, input sanitization, format checking. Example: `validate-json-schema.py`
- **Notification / messaging** — Send alerts via email, Slack, webhooks, or other channels. Example: `send-notification.py`
- **Environment / system info** — Detect OS, available interpreters, installed packages, disk space, running services. Example: `check-environment.py`
- **Credential management** — Load and validate `.env` variables, rotate tokens, check expiration. Example: `load-env.py`
- **Logging / audit** — Structured logging, execution audit trails, progress tracking. Example: `log-event.py`
- **Backup / archive** — Snapshot state, archive outputs, manage retention. Example: `archive-output.py`

When building a new workflow, check `00-TOOLS/core/` first. If a common tool does 80% of what you need, extend it rather than writing a new one from scratch.

## Directory Structure

```
project-root/
├── AGENTS.md                   # Root-level — this file (framework overview)
├── 00-SOPs/
│   ├── AGENTS.md               # Index of all SOPs, conventions, how to write SOPs
│   ├── core/
│   │   ├── AGENTS.md           # What each core SOP does, when to use which
│   │   └── ...
│   └── <workflow-name>/
│       ├── AGENTS.md           # Workflow context, gotchas, lessons learned
│       └── ...
├── 00-TOOLS/
│   ├── AGENTS.md               # Tool inventory, naming conventions, how to add tools
│   ├── core/
│   │   ├── AGENTS.md           # What each shared tool does, inputs/outputs, examples
│   │   └── ...
│   └── <workflow-name>/
│       ├── AGENTS.md           # Workflow-specific tool docs, dependencies, gotchas
│       └── ...
├── 00-KBs/
│   ├── AGENTS.md               # What knowledge is stored here, how to contribute
│   └── ...
├── .tmp/                       # Temporary / intermediate files (gitignored). Everything in `.tmp/` can be deleted and regenerated.
├── .env                        # Credentials and config (gitignored)
└── .env.example                # Documents required env vars (no real values)
```

Use **kebab-case** for all workflow directory names (e.g., `data-pipeline`, `content-generation`).

## Per-Directory AGENTS.md Files

Every folder and subfolder in the project **must** contain its own `AGENTS.md`. These files are the primary way AI agents orient themselves when entering a directory. They are read automatically by AI coding tools, making them the most effective channel for passing knowledge between iterations, agents, and humans.

### What goes in each AGENTS.md

| Directory level | Content |
|----------------|---------|
| **Root** (`./AGENTS.md`) | Framework overview, architecture, global principles (this file) |
| **`00-SOPs/`** | Index of all SOP categories, how to write a new SOP, naming conventions |
| **`00-SOPs/core/`** | Summary of each core SOP, when to use which, cross-references |
| **`00-SOPs/<workflow>/`** | Workflow purpose, context, workflow-specific conventions, lessons learned |
| **`00-TOOLS/`** | Tool inventory overview, naming conventions, how to register a new tool |
| **`00-TOOLS/core/`** | What each shared tool does, its inputs/outputs, usage examples |
| **`00-TOOLS/<workflow>/`** | Workflow-specific tools, their dependencies, integration notes, gotchas |
| **`00-KBs/`** | What knowledge is stored, how to contribute, index of key documents |

### Keeping AGENTS.md files current

AGENTS.md files are **living documents** — they must be updated continuously as the project evolves. Update the relevant AGENTS.md whenever you:

- **Create a new file or folder** — Add an entry describing what it is and when to use it
- **Create or modify a tool** — Document its purpose, inputs, outputs, and any quirks discovered
- **Create or modify an SOP** — Update the directory's index with the new/changed SOP
- **Discover a gotcha or edge case** — Record it immediately so future agents don't repeat the mistake
- **Learn a pattern** — If you find that "X always requires Y", write it down
- **Hit an error and fix it** — Document the root cause and the fix (ties into Self-Annealing Loop)
- **Remove or deprecate something** — Remove stale entries so the file stays trustworthy

### What makes a good AGENTS.md entry

**Good** — specific, actionable, saves future agents real time:
- "When adding a new API tool, always include retry logic — see `api-request.py` for the pattern"
- "`validate-json-schema.py` expects the schema file as the first arg and data on stdin"
- "The `data-pipeline` workflow requires `POSTGRES_URL` in `.env` — will fail silently without it"
- "Files in this folder are auto-loaded by the orchestrator — do not add non-SOP files here"

**Bad** — vague, obvious, or already documented elsewhere:
- "This folder contains tools"
- "Make sure things work correctly"
- "See the README for details" (without specifying which detail)

### Creating AGENTS.md for a new directory

When you create any new folder, immediately create its `AGENTS.md` with at minimum:

1. A one-line description of the directory's purpose
2. A list of its current contents with brief descriptions
3. Any conventions or constraints specific to this directory

This is not optional. An empty directory with no `AGENTS.md` is a dead-end for the next agent that enters it.

## Multi-Workflow Support

Multiple workflows coexist in the same project. When a tool or SOP serves a specific workflow, place it in that workflow's subfolder. When it is useful across workflows, place it at the shared level:

| Scope | SOPs location | TOOLS location |
|-------|--------------|----------------|
| Shared / cross-workflow | `00-SOPs/core/` | `00-TOOLS/core/` |
| Workflow-specific | `00-SOPs/<workflow-name>/` | `00-TOOLS/<workflow-name>/` |

Before creating a new tool or SOP, always check whether a shared one already exists that does the job.

## Tool Language Selection

**Python first.** Use Python as the default language for creating tools.

| Use Python for | Use Bash for |
|---------------|-------------|
| API integrations | Trivial shell one-liners |
| Data processing and transformation | Simple file moves/copies |
| Multi-step logic with error handling | Piping 2-3 CLI commands together |
| Cross-platform portability | System-specific admin tasks with no Python equivalent |
| Anything involving parsing, validation, or complex I/O | |

**System awareness:** Always detect and adapt to the target system. Check the OS (`uname -s`), available Python version (`python3 --version`), and existing tooling before creating or running tools. Write tools that work on the system where the workflow will execute.

## Security

Secure coding is mandatory in every tool. No exceptions.

### Python
- Never use `eval()` or `exec()` on external data
- Use `subprocess.run()` with **list arguments** — never `shell=True` with user-supplied input
- Validate and sanitize all inputs before processing
- Use `pathlib` for file path operations (prevents path traversal)
- Use the `secrets` module for generating tokens or random values
- Use `logging` instead of printing sensitive data

### Bash
- Always start scripts with `set -euo pipefail`
- Quote all variables: `"$var"` not `$var`
- Never use `eval` with user-supplied data
- Validate file paths before operating on them

### JavaScript / Node.js
- Never use `eval()` or `new Function()` with external input
- Sanitize all HTML output to prevent XSS
- Use parameterized queries for any database access
- Validate JSON input against a schema

### General Principles
- **Credentials**: Always in `.env`, always in `.gitignore`. Use `.env.example` to document required variables without values. Never hardcode secrets.
- **Dependencies**: Pin versions. Use well-maintained, widely-adopted packages. Run `pip audit` or `npm audit` regularly.
- **Network**: HTTPS only. Validate TLS certificates. Set timeouts on all requests. Sanitize URLs.
- **Access**: Principle of least privilege. Request only the permissions a tool actually needs.
- **Failure**: Fail closed. When in doubt, deny or stop rather than proceed with partial data.

## Self-Annealing Loop

Errors are not failures — they are improvements waiting to happen. When something breaks:

1. **Read** the error message and understand the root cause
2. **Fix** the tool (if it costs money or uses paid APIs, ask the user first)
3. **Test** the fix to confirm it works
4. **Update the SOP** with what you learned — API constraints, timing issues, edge cases, better approaches
5. **Update the AGENTS.md** in the affected directory — document the gotcha so the next agent doesn't hit it
6. **The system is now stronger** — future runs won't hit the same problem

**Concrete example:** You call an API tool and it fails with a 429 rate limit error. You read the API docs, discover there's a batch endpoint that accepts 100 items per request instead of one-at-a-time. You rewrite the tool to use the batch endpoint with proper rate limiting. You test it. You update the SOP to note the rate limit and the batch approach. You update the directory's AGENTS.md to warn: "This API rate-limits at 60 req/min — always use the batch endpoint." The system is now faster and won't hit this error again.

This loop is the core mechanism that makes SOT workflows improve over time. Never skip steps 4 and 5.

## Operating Principles

1. **Check for existing tools first.** Before writing a new script, look in `00-TOOLS/core/` and the relevant workflow subfolder. Reuse what exists.

2. **SOPs are living documents.** When you discover API constraints, better approaches, common errors, or timing expectations — update the relevant SOP. Improvements compound across every future run.

3. **Never create or overwrite SOPs without asking.** SOPs are the shared instruction set. Propose changes to the user before modifying them.

4. **Keep tools deterministic.** Tools should produce the same output given the same input. Push all complexity into scripts so the AI layer only handles routing and judgment.

5. **Document reusable knowledge.** When you discover patterns, gotchas, or domain insights that don't belong in a specific SOP, add them to `00-KBs/`.

6. **Keep AGENTS.md files current.** Every time you create, modify, or delete a file or folder — update the AGENTS.md in that directory. Every time you learn something — write it down. Stale documentation is worse than no documentation because it misleads.

7. **Automate, don't do manually.** If a task will be done more than once, it should be a tool. Use scripts instead of manual work. The upfront cost of creating a tool pays for itself on every subsequent run.

8. **Intermediates are disposable, deliverables are not.** Local files in `.tmp/` are for processing only — they can be deleted and regenerated at any time. Deliverables (final outputs the user needs) live wherever the user can access them: cloud services, output directories, shared drives, databases, etc. Never treat an intermediate as a deliverable.


## Summary

You sit between human intent (SOPs) and deterministic execution (Python / Bash / PowerShell / JavaScript / etc scripts). Read instructions, make decisions, call tools, handle errors, continuously improve the system.

Be pragmatic. Be reliable. Self-anneal.