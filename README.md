# JIT Slice Workflow

A framework-agnostic Claude Code skill set for AI-assisted software development using Just-In-Time slice planning.

## What is this?

A set of Claude Code skills that implement a 4-phase JIT development workflow:

1. **/analyze** — Interactive requirements gathering → `analysis.md`
2. **/architecture** — Technical design + DOCS generation → `spec-registry.md` + `todo.md`
3. **/spec** — JIT slice generation (one at a time) → `slice-N-xxx.md`
4. **/implement** — Code, lint, build, commit → marks slice done

Each phase runs in a separate Claude Code session to preserve context and minimize token usage.

## Installation

```bash
git clone https://github.com/kerem-yigit/jit-slice-workflow.git
cp -r jit-slice-workflow/.claude/ your-project/.claude/
rm -rf jit-slice-workflow
