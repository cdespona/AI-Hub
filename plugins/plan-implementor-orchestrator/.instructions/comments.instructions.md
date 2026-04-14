---
description: 'Comment and documentation rules for clean, maintainable, agent-friendly Java/Kotlin code'
applyTo: '**/*.{kt,java}'
---

# Comments and Documentation Instructions

Use this file to keep code self-explanatory and comments high-signal.

## Core Principle

Default to **no comments**.  
Prefer better naming, smaller functions, and clearer structure over explanatory text.

## Comment Decision Flow (Mandatory)

Before adding any comment, apply this order:

1. **Refactor first**: Can structure make intent obvious?
2. **Rename second**: Can names communicate the idea?
3. **Delete noise**: If comment explains *what* code does, remove it.
4. **Keep only why**: Add a brief comment only if rationale cannot be expressed in code.

If steps 1-3 solve clarity, do not add a comment.

## Allowed Comments

Add comments only for cases where intent is not obvious from code:

- Non-trivial algorithm choice (and why this approach is needed)
- Business or regulatory constraints not inferable from code
- Workarounds for external bugs/limitations
- Concurrency, performance, or security trade-offs that are intentional

## Forbidden Comments

Do not add comments that:

- Restate obvious behavior
- Narrate line-by-line actions
- Duplicate type names, variable names, or method names
- Explain stale behavior after code changed
- Use vague placeholders (`TODO` without owner/date/context)

## Public API Documentation

Use KDoc/Javadoc for **public** APIs only (interfaces, public methods, library-facing contracts).

Document:

- Purpose and behavioral contract
- Preconditions and postconditions
- Important side effects
- Thrown exceptions (when part of contract)

Do not add doc comments to internal/private code unless explicitly required by project policy.

## Comment Quality Rules

When a comment is necessary:

- Keep it short (1-3 lines when possible)
- Explain **why**, not **what**
- Tie it to a concrete constraint (business rule, bug ID, RFC, ticket)
- Keep wording precise and testable

Good format examples:

- `// Required by PCI rule: mask PAN before persistence`
- `// WORKAROUND(#4521): remove after upgrading library >= 2.4.0`
- `// Binary search is safe here because list is sorted by timestamp upstream`

## TODO / FIXME Governance

Use action comments sparingly and only with metadata:

- `TODO(owner, yyyy-mm-dd): short action`
- `FIXME(owner, ticket): short issue`

Rules:

- No anonymous TODO/FIXME
- Remove resolved items immediately
- Convert long-lived TODOs into tracked tickets

## Maintenance Rules (Required on Every Edit)

When touching a file:

1. Remove comments that became redundant
2. Update comments that no longer match behavior
3. Replace explanatory comments with clearer code where feasible
4. Keep domain rationale comments if they still add non-obvious context

## Agent-Specific Behavior

- Do not add comments as a substitute for unclear code.
- If clarity requires a large explanatory comment, refactor instead.
- If uncertain whether rationale is durable, prefer no comment and improve naming.
- Ask for user confirmation only when the comment encodes a business decision that is ambiguous.

## Quick Review Checklist

- [ ] Any comment explains **why** (not **what**)
- [ ] No obvious or stale comments remain
- [ ] Public API docs exist where contract clarity is needed
- [ ] TODO/FIXME comments include owner + traceable context
- [ ] Code readability improved first; comments used only as last resort
