---
name: hermes
description: Use when software has been built and needs audience-specific documentation generated — internal docs (CS/Support), external docs (users/developers), or marketing docs (product marketing/sales). Translates code understanding into documentation for three distinct audiences from a single source.
---

# Hermes

*The messenger who speaks every audience's language.*

## Overview

Hermes generates audience-specific documentation from a single codebase. It analyzes code, docs, specs, git history, and optionally Carto's indexed knowledge to produce structured documentation for three audiences: internal teams (CS/Support), external users (developers/end users), and marketing (product marketing/sales).

**Core principle:** Same source, different lens. Every audience gets fundamentally different documents — different voice, different structure, different level of detail.

**Five modes:**
- **Generate** — full project documentation for all or selected audiences
- **Update** — change-scoped delta since last generation
- **Internal** — internal/CS docs only
- **External** — external/user docs only
- **Marketing** — marketing/sales docs only

## Mode Detection

| Invocation Pattern | Mode |
|-------------------|------|
| "Generate docs for X" / "Document this project" | Generate (all audiences) |
| "Generate internal docs" / "CS documentation" | Internal only |
| "Generate external docs" / "API docs" / "User docs" | External only |
| "Generate marketing docs" / "Feature briefs" / "Datasheet" | Marketing only |
| "Update docs" / "Document what changed" / "Release docs" | Update (delta) |
| "Generate and publish docs" | Generate, then publish |
| No docs exist + user says "document this" | Generate (full) |

**Default behavior:** If `docs/generated/` doesn't exist, always Generate full. If it exists and user says "update" or references a version/release, use Update mode.

## Resume Protocol

Every Hermes invocation starts here. Check disk state before doing any work.

1. Check if `docs/generated/.hermes.md` exists
2. **If no:** Fresh run — proceed to Input Resolution (Step 1)
3. **If yes:** Read `.hermes.md` — check generation state
   - If docs are partially generated (some audiences pending): resume from next pending audience
   - If all audiences are complete and user asked for update: proceed to Update mode
   - If all audiences are complete and user asked for generate: ask "Docs already exist. Regenerate from scratch or update with changes?"

**Key rule:** Never re-generate completed docs unless explicitly asked. Check disk state, skip completed work.

## Document Templates

### Internal: Feature Handoff

**File:** `docs/generated/internal/feature-handoffs/fh-{NNN}-{slug}.md`

~~~markdown
---
id: fh-{NNN}
type: feature-handoff
audience: internal
topic: {feature-name}
status: draft
generated: {date}
source-tier: {carto|direct|context}
context-files: [{list}]
hermes-version: 0.1.0
---

# Feature Handoff: {Feature Name}

## What It Does
{1-2 paragraph summary — what this feature enables, who uses it, when they use it}

## How It Works
{Technical explanation of the mechanism — components involved, data flow, key logic. Enough detail for L2 support to understand root causes, not just symptoms.}

## User-Facing Behavior
{What the end user sees and experiences:
- UI changes (new screens, modified flows, new buttons/options)
- API changes (new endpoints, changed responses, new parameters)
- Behavior changes (different defaults, new notifications, changed workflows)}

## Configuration
{Every config option, feature flag, or environment variable that affects this feature:
| Option | Type | Default | Effect |
|--------|------|---------|--------|
| ... | ... | ... | ... |}

## Edge Cases & Limitations
{Known limitations, boundary conditions, what it explicitly doesn't handle. Be specific:
- "Does NOT support batch operations over 1000 items"
- "Requires user to have admin role — other roles see a 403"}

## Common Questions
{Anticipated questions from CS/Support — the ones they'll get from customers:
**Q: {question}**
A: {answer}

Repeat for 3-5 most likely questions.}

## Troubleshooting
{Known error scenarios and diagnostic steps:
| Symptom | Likely Cause | Resolution |
|---------|-------------|------------|
| ... | ... | ... |}

## Related
{Links to related feature handoffs, design docs, external docs}
~~~

### Internal: Release Notes

**File:** `docs/generated/internal/release-notes/rn-v{version}.md`

~~~markdown
---
id: rn-{version}
type: release-notes
audience: internal
version: {version}
status: draft
generated: {date}
commit-range: {from-hash}..{to-hash}
source-tier: {tier}
hermes-version: 0.1.0
---

# Internal Release Notes — v{version}

**Date:** {date}
**Commits:** {count} since v{previous-version}
**Contributors:** {list of commit authors}

## Summary
{2-3 sentence summary of what this release contains — lead with the most important change}

## Breaking Changes
{List each breaking change with:
- What changed
- Why it changed
- Migration steps (exact commands or code changes needed)
- Who is affected (which teams, which customers, which integrations)}

## New Features
{For each new feature:
### {Feature Name}
- **What:** {1-sentence description}
- **How:** {brief technical explanation}
- **Config:** {new config options or feature flags}
- **Who it affects:** {which user segments, teams, or workflows}
- **CS Notes:** {what CS/Support needs to know that's different from the external changelog}}

## Improvements
{Enhancements to existing features — same structure as new features but briefer}

## Bug Fixes
{For each fix:
- **Symptom:** {what users were experiencing}
- **Root Cause:** {what was wrong}
- **Resolution:** {what was fixed}
- **Tickets:** {related ticket IDs if available}}

## Configuration Changes
{Table of new, changed, or removed config options:
| Option | Change | Old Value | New Value | Notes |
|--------|--------|-----------|-----------|-------|
| ... | added/changed/removed | ... | ... | ... |}

## Known Issues
{Issues shipping with this release — be honest:
- {issue description} — workaround: {workaround if any}}

## Internal Notes
{Anything not in the above that internal teams need to know — deployment notes, rollout plan, feature flag states}
~~~

### External: Changelog

**File:** `docs/generated/external/changelog.md`

Generated from git history. Groups commits by type. Translates engineering language to user-facing language.

~~~markdown
---
type: changelog
audience: external
generated: {date}
hermes-version: 0.1.0
---

# Changelog

All notable changes to this project are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/).

## [{version}] - {date}

### Added
- {user-facing description of new features — benefit-focused, not implementation-focused}

### Changed
- {modifications to existing behavior — what's different for the user}

### Deprecated
- {features marked for future removal — include timeline and replacement}

### Removed
- {features removed — include migration path}

### Fixed
- {bug fixes — describe the symptom that's resolved, not the code change}

### Security
- {security fixes — enough detail to understand severity, not enough to exploit}
~~~

**Generation rules for changelog:**
1. Parse git log for commits since last generation (or all commits for initial)
2. If conventional commits: group by type prefix (feat→Added, fix→Fixed, etc.)
3. If freeform commits: analyze commit messages semantically to categorize
4. Translate every entry from engineering language ("fixed null pointer in auth middleware") to user language ("Fixed an issue where login could fail intermittently")
5. Omit purely internal changes (refactors, CI changes, dependency bumps) unless they affect user behavior
6. List breaking changes prominently with migration guidance

### Marketing: Feature Brief

**File:** `docs/generated/marketing/feature-briefs/fb-{NNN}-{slug}.md`

This is the **keystone document** — other marketing docs derive from it.

~~~markdown
---
id: fb-{NNN}
type: feature-brief
audience: marketing
topic: {feature-name}
status: draft
generated: {date}
source-tier: {tier}
context-files: [{list}]
hermes-version: 0.1.0
---

# Feature Brief: {Feature Name}

## One-Liner
{Single sentence — what it is and why it matters. Usable as a tagline or email subject.}

## What It Is
{2-3 sentences in non-technical language. A product marketer should be able to read this and immediately understand the feature without any engineering background.}

## Who It's For
{Target personas or segments. Be specific:
- **Primary:** {who benefits most — role, company size, use case}
- **Secondary:** {who else benefits}}

## The Problem It Solves
{In the customer's language — what pain they feel today that this feature addresses. Frame as a story, not a spec:
"Teams currently have to manually X every time they Y, which takes Z hours and often leads to errors..."}

## Key Benefits
{3-5 benefits — outcome-focused, not feature-focused:
- **{Benefit headline}:** {1-sentence elaboration}
Repeat for each benefit.}

## How It Works (Simplified)
{Non-technical explanation. Enough for marketing to understand and explain, not enough to implement. Use analogies if helpful. 3-5 sentences max.}

## Competitive Context
{How this compares to alternatives — only populated if context files provide competitive info:
- **vs. {Competitor}:** {our advantage}
If no competitive context available, state: "Competitive positioning requires additional context — provide competitive analysis docs to populate this section."}

## Proof Points
{Metrics, benchmarks, or evidence of value. Only populated if available from context:
- {proof point}
If none available, state: "Proof points require usage data or customer feedback to populate."}

## Suggested Messaging
{2-3 ready-to-use messages for different contexts:
- **Announcement:** {for blog post, email, or social media}
- **Sales pitch:** {for sales conversations}
- **One-liner:** {for landing pages or feature lists}}
~~~

### Marketing: Product Datasheet

**File:** `docs/generated/marketing/datasheet.md`

~~~markdown
---
id: ds-001
type: datasheet
audience: marketing
status: draft
generated: {date}
source-tier: {tier}
hermes-version: 0.1.0
---

# {Product Name} — Product Datasheet

## Overview
{2-3 sentence product description — what it is, who it's for, primary value proposition}

## Key Capabilities
{Bulleted list of major features, grouped by category if >10 items:
### {Category}
- {Capability}: {1-sentence description}}

## Technical Specifications
| Spec | Value |
|------|-------|
| Language/Runtime | {from package manifest} |
| Supported Platforms | {from code/config} |
| API Style | {REST/GraphQL/gRPC/CLI — from code} |
| Authentication | {auth methods — from code} |
| Data Storage | {database type — from config/code} |

## Dependencies
{Key runtime dependencies — not dev dependencies:
| Dependency | Version | Purpose |
|-----------|---------|---------|
| ... | ... | ... |}

## Integrations
{List of supported integrations, derived from code:
- **{Integration}:** {what it enables}}

## Deployment Options
{How it can be deployed:
- {option}: {brief description}}

## Security & Compliance
{Security features derived from code — auth methods, encryption, audit logging.
Compliance certs only if documented in context files.}

## System Requirements
{Minimum requirements from docs, config, or Dockerfile:
| Requirement | Minimum |
|------------|---------|
| ... | ... |}
~~~

---

## Step-by-Step Process

### Step 1: Input Resolution

Resolve input sources in order. Announce what's available.

**1a. Check for Carto index:**
- Search Memories for source prefix `carto/{project-name}/`
- If found: pull entries for `layer:atoms`, `layer:blueprint`, `layer:zones`, `layer:patterns`
- Announce: "Found Carto index for {project} — using full code understanding."
- Set source-tier to `carto`

**1b. Fallback to direct codebase (if no Carto):**
- Glob for key files:
  - Project manifest: `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `pom.xml`
  - Documentation: `README.md`, `CLAUDE.md`, `CONTRIBUTING.md`
  - Config: `.apollo.yaml`, `*.config.*`, `.env.example`
  - Source directories: `src/`, `lib/`, `app/`, `api/`, `cmd/`, `pkg/`
  - API specs: `openapi.yaml`, `openapi.json`, `swagger.*`
- Read key files (README, CLAUDE.md, manifest, main entry points)
- Run: `git log --oneline -30` for recent history
- Run: `git tag -l --sort=-version:refname | head -5` for recent versions
- Announce: "No Carto index — reading codebase directly."
- Set source-tier to `direct`

**1c. Context file discovery (always runs):**
- Auto-discover:
  - `docs/plans/*.md` — design documents
  - `docs/decisions/*.md` — decision records
  - `.apollo.yaml` — project config
  - `CHANGELOG.md` — existing changelog
  - `docs/VISION.md` — project vision
- User-specified files: if user mentioned specific files, read those too
- Announce: "Also using {N} context files: {file list}"

**1d. Change detection (Update mode only):**
- Read `.hermes.md` for last generation timestamp
- Run: `git log --oneline --since="{timestamp}"` to find changes
- Run: `git diff --stat {last-commit}..HEAD` to see what files changed
- Identify affected features/components from changed files
- Announce: "{N} commits since last generation affecting {list of areas}"

### Step 2: Feature/Component Discovery

From gathered input, identify what to document.

**If Carto is available (Tier 1):**
- Use Carto zones as the feature grouping — each zone is a documentable feature area
- Use Carto atoms within each zone for detailed behavior
- Use Carto blueprint for the system overview

**If direct codebase (Tier 2):**
- Scan source directories for logical groupings:
  - Route files → user-facing features
  - API handlers → API surface
  - Models/schemas → data entities
  - Config files → configurable behavior
  - CLI commands → CLI surface
- Group related files into feature areas by directory structure and naming
- Read key files to understand purpose

**For both tiers:**
- Build a feature map:
  ```
  Feature: {name}
  Components: {list of files/modules}
  Surface: {ui|api|cli|library|service}
  Description: {1-sentence summary}
  ```

**Present the feature map to the user:**

> **Feature Map:**
>
> **authentication** (4 components)
> - `src/auth/` — login, registration, session management
> - Surface: UI + API
>
> **dashboard** (3 components)
> - `src/dashboard/` — stats, activity feed, widgets
> - Surface: UI
>
> Should I generate docs for all features, or focus on specific ones?

**Wait for user confirmation before proceeding.** The user may scope down.

**Write the state file:**

After user confirms, write `docs/generated/.hermes.md`:

```markdown
# Hermes State

## Configuration
- Source tier: {carto|direct}
- Context files: {list}
- Hermes version: 0.1.0

## Features
| Feature | Components | Status |
|---------|-----------|--------|
| {name} | {count} files | pending |

## Generation Progress
- Internal: pending
- External: pending
- Marketing: pending
- Last generated: never
```

### Step 3: Generate Documentation

**Chunking: one audience at a time. Features within each audience can be parallel.**

Process audiences in this order: Internal → External → Marketing. This order matters because:
1. Internal docs force you to understand the feature deeply (edge cases, troubleshooting)
2. External docs build on that understanding but simplify for users
3. Marketing docs build on both but reframe for value

**For each audience (or only the selected audience if single-audience mode):**

**3a. Create output directory structure:**
```bash
mkdir -p docs/generated/internal/feature-handoffs
mkdir -p docs/generated/internal/release-notes
mkdir -p docs/generated/external
mkdir -p docs/generated/marketing/feature-briefs
```

**3b. Generate docs for each confirmed feature:**

For each feature in the feature map:

**Internal — Feature Handoff:**
1. Gather all context for this feature (Carto atoms or source files + context docs)
2. Generate the feature handoff using the template above
3. Voice: practical, scenario-driven. Write for a CS agent who needs to answer customer questions.
4. Be SPECIFIC — name exact config options, exact error messages, exact UI elements
5. The "Common Questions" section is critical — anticipate what customers will ask
6. Write file to `docs/generated/internal/feature-handoffs/fh-{NNN}-{slug}.md`

**External — Feature Docs (changelog handled separately):**
- For full generate mode: generate feature-level docs describing user-facing behavior
- Voice: clear, second-person ("You can...", "To configure X, do Y")
- Focus on what the user can DO, not how it works internally
- Write to `docs/generated/external/features/feat-{NNN}-{slug}.md` (Phase 2)

**Marketing — Feature Brief:**
1. Gather feature context + any business context from context files
2. Generate the feature brief using the template above
3. Voice: value-driven, benefit-first. Write for a product marketer.
4. Translate every technical capability into a user benefit
5. The "Suggested Messaging" section should be ready-to-use copy
6. Write file to `docs/generated/marketing/feature-briefs/fb-{NNN}-{slug}.md`

**3c. Generate cross-feature docs:**

**Internal Release Notes** (if version context is available):
1. Parse git log for the relevant commit range
2. Categorize commits by type and affected feature
3. Generate internal release notes using the template
4. Write to `docs/generated/internal/release-notes/rn-v{version}.md`

**External Changelog:**
1. Parse same git log
2. Translate to user-facing language
3. Follow Keep a Changelog format
4. Write to or append `docs/generated/external/changelog.md`

**Marketing Product Datasheet:**
1. Aggregate all features, specs, and capabilities
2. Extract technical specs from manifests, configs, Dockerfiles
3. Generate datasheet using the template
4. Write to `docs/generated/marketing/datasheet.md`

**3d. After each audience is complete:**
- Update `.hermes.md` — mark audience as `done`, update timestamp
- If single-audience mode: stop here

### Step 4: Generate Index

Write `docs/generated/index.md`:

```markdown
# Hermes Documentation Index

Generated: {timestamp} | Source: {tier} | Hermes v0.1.0

## Internal ({count} docs)
| ID | Type | Topic | Generated |
|----|------|-------|-----------|
| fh-001 | Feature Handoff | {topic} | {date} |
| rn-v{ver} | Release Notes | v{ver} | {date} |

## External ({count} docs)
| ID | Type | Topic | Generated |
|----|------|-------|-----------|
| changelog | Changelog | All | {date} |

## Marketing ({count} docs)
| ID | Type | Topic | Generated |
|----|------|-------|-----------|
| fb-001 | Feature Brief | {topic} | {date} |
| ds-001 | Datasheet | Full Product | {date} |
```

### Step 5: Store in Memories

For each generated document, store a summary in Memories for AI consumption:

```
memory_add(
  source: "hermes/{project}/internal",
  text: "FEATURE-HANDOFF: {feature}. {2-sentence summary}. Generated: {date}. File: {path}."
)

memory_add(
  source: "hermes/{project}/marketing",
  text: "FEATURE-BRIEF: {feature}. ONE-LINER: {one-liner}. KEY-BENEFITS: {benefits}. Generated: {date}. File: {path}."
)

memory_add(
  source: "hermes/{project}/state",
  text: "GENERATION: {date}. Source: {tier}. Features: {count}. Internal: {count} docs. External: {count} docs. Marketing: {count} docs."
)
```

### Step 6: Finalize

Update `.hermes.md` with final state:

```markdown
## Generation Progress
- Internal: done ({count} docs)
- External: done ({count} docs)
- Marketing: done ({count} docs)
- Last generated: {timestamp}
- Last commit: {hash}
- Source tier: {tier}
```

Report to user:

> **Hermes Documentation Generated**
>
> Source: {tier} | Features: {count}
>
> | Audience | Docs | Path |
> |----------|------|------|
> | Internal | {N} | `docs/generated/internal/` |
> | External | {N} | `docs/generated/external/` |
> | Marketing | {N} | `docs/generated/marketing/` |
>
> Review the generated docs, then commit when ready.

---

## Update Mode

For change-scoped documentation (documenting what changed since last generation).

### Step 1: Detect Changes
1. Read `.hermes.md` for last generation timestamp and commit hash
2. Run `git log --oneline {last-commit}..HEAD` to find new commits
3. Run `git diff --name-only {last-commit}..HEAD` to find changed files
4. Map changed files to features using the feature map

### Step 2: Scope
- Only generate docs for features that have changed files
- Announce: "{N} features changed since last generation: {list}"

### Step 3: Generate Deltas
For each changed feature:
- **Internal:** Generate or update feature handoff. If it exists, update the relevant sections.
- **Internal:** Generate new release notes for the new version/commit range
- **External:** Append new entries to changelog
- **Marketing:** Generate new feature brief only for NEW features. Update existing briefs for modified features.

### Step 4: Update State
- Update `.hermes.md` with new timestamp and commit hash
- Update index.md with new/modified entries
- Store updated summaries in Memories

---

## Parallel Dispatch

When multiple features need documentation, dispatch subagents for parallel generation.

**Main agent responsibilities (NEVER delegated):**
- Input resolution (Step 1)
- Feature discovery (Step 2)
- State file management (.hermes.md)
- Index generation (Step 4)
- Memory storage (Step 5)
- Final reporting (Step 6)

**Subagent responsibilities (one per feature, per audience):**
- Receive: feature name, source files/Carto data, context files, template, voice guidelines
- Generate: one document per the template
- Write: to the correct output path
- Return: summary of what was generated

**Subagent prompt must include:**
1. The complete template for the doc type being generated
2. Voice guidelines for the audience
3. All source data (file contents, Carto entries) — subagents cannot search Memories
4. Output file path
5. Generation rules

**After all subagents complete:**
1. Verify all expected files exist on disk
2. Generate the index
3. Update .hermes.md
4. Store in Memories

---

## Conductor Integration

Hermes slots into Conductor's pipeline as a `communicate` phase:

```yaml
skills:
  hermes:
    source: external
    phase: communicate
    type: phase

pipelines:
  feature:
    phases: [explore, plan, build, verify, communicate, review, finish]
    skills:
      communicate:
        - hermes
```

When triggered by Conductor:
- Mode defaults to `update` (document what was just built)
- Scope automatically limited to files changed in the current pipeline
- Runs after verify (Delphi), before review

---

## Voice Guide

### Internal Voice
- **Tone:** Practical, direct, thorough
- **Perspective:** Third person ("The system does X") or instructional ("Check the config for Y")
- **Assumption:** Reader is technical enough to understand system behavior but may not know THIS specific feature
- **Priority:** Accuracy over elegance. Be specific: name exact config keys, exact error messages, exact API responses
- **Anti-pattern:** Don't say "the feature works as expected" — describe WHAT it does specifically

### External Voice
- **Tone:** Clear, helpful, encouraging
- **Perspective:** Second person ("You can...", "To do X, run Y")
- **Assumption:** Reader may be new to the project. Don't assume knowledge of internals.
- **Priority:** Clarity over completeness. Lead with the most common use case.
- **Anti-pattern:** Don't expose internal architecture ("the middleware validates your token") — say "your request is authenticated automatically"

### Marketing Voice
- **Tone:** Confident, benefit-focused, concise
- **Perspective:** Product-centric ("Now you can...", "{Product} enables...")
- **Assumption:** Reader cares about outcomes, not mechanisms
- **Priority:** Value over detail. Every sentence should answer "so what?"
- **Anti-pattern:** Don't describe implementation ("built with React and PostgreSQL") — describe value ("instant, responsive interface that handles millions of records")

---

## Quality Checks

Before finalizing any document, verify:

1. **Frontmatter is complete** — all required fields populated
2. **No placeholder text** — no `{TODO}`, `TBD`, or `[fill in]` remaining
3. **Voice matches audience** — internal docs aren't marketing-speak, marketing docs aren't technical jargon
4. **Cross-references are valid** — links to other docs point to files that exist
5. **Feature Handoff has Common Questions** — never empty, minimum 3 Q&A pairs
6. **Changelog entries are user-facing** — no internal refactors or CI changes unless they affect users
7. **Marketing briefs have Suggested Messaging** — never empty, minimum 2 messages
8. **Datasheet specs are factual** — only include what's verifiable from code/config, not aspirational
