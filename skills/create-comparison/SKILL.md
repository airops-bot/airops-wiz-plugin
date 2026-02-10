---
name: create-comparison
description: Create a comparison article (X vs Y) that helps readers choose.
argument-hint: <subject_a> <subject_b> [audience=general] [bias=neutral] [word_count=2000]
disable-model-invocation: true
---

Create a comparison article: $ARGUMENTS

**Steps:**

1. [Research](1-research/)
2. [Outline](2-outline/)
3. [Draft](3-draft/)
4. [Finalize](4-finalize/)

**Inputs:** subject_a, subject_b (required), audience (default general), bias (default neutral), word_count (default 2000). Resolve from $ARGUMENTS; replace `{subject_a}`, `{subject_b}`, `{audience}`, `{word_count}` in step instructions.

---

# Run context (orchestration)

**How to execute a playbook.** Step instructions live in that skill's step folders (e.g. `1-research/`, `2-outline/`). Load each step's instructions only when you reach that step.

**At the start of every run:** Create a todo list with one item per step (in the order listed in the playbook's SKILL.md). Work through the steps one by one; complete a step fully and mark its todo complete before starting the next. Do not skip or reorder steps.

**Agents:** For every **writing** step (draft, write, edit, refine, finalize), use the **content-writer** agent — do not execute writing in the main session. For the final content step, also use the **content-critic** agent to validate. Use **ai-search-analyst** for AI-search/AEO data interpretation and **page-analyst** for page-level diagnosis when those data sources are available.

---

## Phase 1: Parse arguments

Parse: optional `--session=...`; then positionals and `key=value` pairs. Map them to this playbook's **inputs** (from the skill frontmatter): first positional → first required input; named args → matching input names; missing optionals → defaults. Missing required → error with usage.

### Artifact Contract (required)

- Resolve `SESSION_DIR`:
  - if `--session=<id>` is provided: `SESSION_DIR = sessions/<id>/`
  - otherwise create one: `SESSION_DIR = sessions/<generated-id>/`
- All step outputs must be written under `SESSION_DIR`.
- Never write artifacts to repo root.
- Overwrite in place; no timestamped filenames.
- If `session_id` cannot be resolved, stop and request clarification.

Canonical artifact paths:
- `sessions/{session_id}/research.md`
- `sessions/{session_id}/outline.md`
- `sessions/{session_id}/draft.html`
- `sessions/{session_id}/final.html`

---

## Phase 2: Brand context

Before executing any steps, ensure brand context is understood. Use whatever tools are available (search_brand_context, get_brand_kit_item, or read files if in plugin). Do not invent brand voice or terminology.

If a step includes a Brand Kit reference token like `{{@brandkit:type/name.md}}`, treat it as a human-readable pointer. Use `search_brand_context` with the referenced name/type to find the item content before continuing.

### Custom MCP tools (when available)

- `ReportStepProgress` for step lifecycle reporting.
- `PauseForUserInput` for checkpoints.
- `get_brand_context_filters` + `search_brand_context` + `get_brand_kit_item` for brand navigation.
- `get_knowledge_base_filters` + `SearchKnowledgeBase` + `get_knowledge_base_item` for sources/facts.
- `skills/aeo-data-guide/SKILL.md` as the shared reference for interpreting AI-search/AEO metrics.

---

## Phase 3: Execute each step

Work through the playbook's step list **in order**. For each step:

1. Ensure the step is in your todo list; start the step (e.g. mark in progress if your UI supports it).
2. Load that step's instructions from `skills/{playbook}/{n}-{name}/instructions.md`, execute (or delegate to the right agent), save artifacts into `SESSION_DIR`, then mark the step todo complete.
3. Only then move to the next step.

If your environment provides `ReportStepProgress`, call it when starting and completing each step with step number/title/status and artifact paths under `SESSION_DIR`.

### Execute

- **Research steps**: Run the step instructions in this session. Use available tools (web search, knowledge base, brand context).
- For AEO/AI-search heavy research, delegate data interpretation to `ai-search-analyst` and synthesize results before writing.
- **Writing steps**: Delegate to the **content-writer** subagent. Give it the step instructions (with variables resolved), instruction to load brand context first, references to artifacts produced so far, requirement to apply the seo-guidelines skill, and where to save the output.

Save artifacts after each step in `SESSION_DIR`. Use sensible names from the step (e.g. `research.md`, `outline.md`, `draft.html`, `final.html`).

### Validate (final writing step only)

Only for the **last** content-producing step: run the **content-critic** subagent on the output. If the critic reports issues: have content-writer fix all issues in one pass, re-validate once, and at most two fix iterations — then proceed and note any remaining issues.

---

## Phase 4: Checkpoints

If a step mentions a checkpoint or human review, complete the step, then use a pause-for-input tool if available and stop until the user responds.

---

## Phase 5: Completion

Summarize: where artifacts live, list of outputs, and the final deliverable.
