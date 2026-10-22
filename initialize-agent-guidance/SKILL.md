---
name: initialize-agent-guidance
description: Create or refresh repository guidance for coding agents by producing an evidence-based AGENTS.md and .github/copilot-instructions.md. Use when onboarding a repository to AI coding agents, initializing Copilot instructions, or standardizing agent workflow guidance.
---

# Initialize Agent Guidance

Create concise, repository-specific guidance. Treat existing guidance as user-owned source material, not boilerplate to replace.

## Inspect First

1. Locate existing `AGENTS.md`, `.github/copilot-instructions.md`, contribution docs, READMEs, and nearby instructions.
2. Inspect the project layout and the authoritative build, test, lint, format, typecheck, and development entrypoints.
3. Infer conventions only from repository evidence: manifests, configuration, CI, source patterns, and existing documentation.
4. Ask before making a material policy choice that cannot be inferred safely.

## Create the Canonical Agent Guide

Create or update root `AGENTS.md`. Keep it tool-neutral and make it the detailed, canonical guide for coding agents. Cover only supported facts:

- architecture and important directories;
- setup and native development, build, test, lint, format, and typecheck commands;
- coding, naming, and test conventions;
- generated files, migrations, secrets, lockfiles, and other safety boundaries;
- expected change scope and validation before handoff.

Preserve useful existing material and remove contradictions or duplicated generic advice. Do not invent commands, rules, owners, or architecture.

## Create the Copilot Companion

Create or update `.github/copilot-instructions.md` (with the leading dot). Keep it brief and refer to `AGENTS.md` as the source of truth. Include only Copilot-specific execution behavior:

- inspect local patterns before editing;
- make small, scoped changes;
- use native project tooling;
- run relevant validation;
- summarize changed files, validation, and caveats;
- never disclose secrets or modify generated/vendor content unless required.

Do not repeat the full `AGENTS.md`. If the repository already has a `github/copilot-instructions.md` without the leading dot, preserve it unless the user explicitly asks to migrate it; Copilot's supported repository location is `.github/copilot-instructions.md`.

## Finish

1. Verify both files are valid Markdown, correctly located, concise, and mutually consistent.
2. Run a relevant non-mutating validation if the repository provides one; otherwise report that no repository-specific validation applies.
3. Report the files created or updated, evidence used, validations run, and anything needing maintainer confirmation.
