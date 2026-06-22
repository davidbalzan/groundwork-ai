---
name: plan-phase
description: Generate a new phase task document using the project template
argument-hint: "<phase number> <phase name>"
---

# Plan Phase - Task Document Generator

Generate a comprehensive phase task document following the Groundwork methodology.

## Obsidian Format (Required)

Generated phase docs live in the `docs/` Obsidian vault. Both files MUST:

1. **Start with YAML frontmatter.** For `docs/phases/phaseN/README.md`:
   ```yaml
   ---
   title: "Phase N: [Name]"
   tags: [groundwork/phase]
   aliases: ["Phase N"]
   ---
   ```
   For `docs/phases/phaseN/PHASEN_TASKS.md`:
   ```yaml
   ---
   title: "Phase N Tasks"
   tags: [groundwork/phase, groundwork/tasks]
   aliases: ["Phase N Tasks"]
   ---
   ```
2. **Use `[[wikilinks]]` for cross-references.** Link back to `[[PRODUCTION_ROADMAP]]`, `[[ARCHITECTURE_GUIDE]]`, `[[TECH_STACK]]`, `[[DECISIONS]]`, `[[CURRENT_FOCUS]]`, and sibling phase files as relevant.
3. **Base the phase README on `docs/templates/PHASE_README_TEMPLATE.md`** and the tasks file on `docs/phases/templates/TASK_TEMPLATE.md`. Both templates already have frontmatter — preserve the structure and update titles.

## Instructions

This skill follows the 6-stage approach from `docs/phases/templates/task_template_prompt.md`:

### Stage 1: Initial Discovery & Project Scope

- Read existing phase documentation
- Understand the phase goals and context
- Review PRODUCTION_ROADMAP.md for phase objectives

### Stage 2: Deep Technical Analysis

- Explore relevant codebase areas
- Identify existing patterns to follow
- Review ARCHITECTURE_GUIDE.md for constraints
- Check TECH_STACK.md for technology requirements

### Stage 3: Impact & Risk Analysis

- Identify dependencies on other phases
- Assess risks and mitigation strategies
- Note potential blockers

### Stage 4: Detailed Component Inventory

- List all components to create/modify
- Map to packages (client/server)
- Identify shared types and interfaces

### Stage 5: Task Planning & Sequencing

- Break into 4-6 major tasks
- Each task gets 3-7 sub-steps
- Assign priorities (CRITICAL/HIGH/MEDIUM/LOW)
- Define dependencies between tasks

### Stage 6: Success Criteria & Validation

- Define functional success criteria
- Define technical success criteria
- Define quality success criteria

## Template Structure

Use `docs/phases/templates/TASK_TEMPLATE.md` as the base.

Create files:

- `docs/phases/phaseN/README.md` - Phase overview
- `docs/phases/phaseN/PHASEN_TASKS.md` - Detailed tasks

## Task Format

Each task should include:

```markdown
### Task N: [Task Name]

**Priority**: [CRITICAL/HIGH/MEDIUM/LOW]
**Package**: [client/server/shared]
**Dependencies**: [Task numbers or "None"]

#### Sub-tasks

- [ ] N.1 [First sub-task]
- [ ] N.2 [Second sub-task]
- [ ] N.3 [Third sub-task]

#### Deliverables

- [What this task produces]

#### Rollback Plan (if high-risk)

- [How to revert if needed]
```

## Output

1. Generate the phase README
2. Generate the detailed tasks file
3. Update PRODUCTION_ROADMAP.md to include the new phase
4. Update docs/BACKLOG.md (see below)
5. Summarize what was created

## BACKLOG.md Integration

After generating the phase task file, update `docs/BACKLOG.md`. This file is the inbound queue for the coordinator-playbook multi-agent execution loop — it is the bridge between Groundwork phase planning and live agent dispatch.

### File format

```markdown
# BACKLOG — <project>

## Queue

- [ ] (P1) Phase N: [Phase Name] — see [[phases/phaseN/PHASEN_TASKS]] for full breakdown · acceptance: all Phase N success criteria met
- [ ] (P2) <ad-hoc task added by David>

## Done

- [x] <task> — owner/repo#N · YYYY-MM-DD
```

### Rules

- **Each phase maps to a single Queue item** — reference the PHASEN_TASKS file rather than listing sub-tasks inline. The coordinator reads the linked file for the full contract.
- **Priority mapping**: CRITICAL phase → P1, HIGH → P2, MEDIUM/LOW → P3. Default new phases to P1 unless the roadmap says otherwise.
- **If docs/BACKLOG.md does not exist**, create it from this template. If it exists, append the new phase item to the `## Queue` section — never overwrite or reorder existing items (David owns that region).
- **Ad-hoc tasks** (bug fixes, improvements David adds directly) live alongside phase items in the Queue — do not remove or reorder them.
- **Do not touch `## Done`** — that section is written by the coordinator only.

Phase to plan: $ARGUMENTS
