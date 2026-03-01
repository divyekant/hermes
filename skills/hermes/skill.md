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
hermes-version: 1.0.0
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
hermes-version: 1.0.0
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
hermes-version: 1.0.0
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
hermes-version: 1.0.0
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
hermes-version: 1.0.0
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

### Internal: Use Case Docs

**File:** `docs/generated/internal/use-cases/uc-{NNN}-{slug}.md`

~~~markdown
---
id: uc-{NNN}
type: use-case
audience: internal
topic: {scenario-name}
status: draft
generated: {date}
source-tier: {tier}
hermes-version: 1.0.0
---

# Use Case: {Scenario Title}

## Trigger
{What initiates this scenario — user action, system event, scheduled task, API call}

## Preconditions
{What must be true before this scenario can occur:
- User must be authenticated
- Feature flag X must be enabled
- Data must exist in Y}

## Flow
{Step-by-step description of what happens — from the user's perspective AND the system's:
1. **User does:** {action}
   **System does:** {response}
2. **User sees:** {result}
   **System does:** {background processing}}

## Variations
{Alternative paths through this scenario:
- **If {condition}:** {different behavior}
- **If {error condition}:** {error handling behavior}}

## Edge Cases
{Boundary conditions and unusual situations:
- {edge case}: {what happens}
- {edge case}: {what happens}}

## Data Impact
{What data is created, modified, or deleted during this flow:
| Data | Action | Location |
|------|--------|----------|
| ... | created/updated/deleted | ... |}

## CS Notes
{Specific guidance for customer-facing teams:
- Common customer confusion points
- How to explain this to non-technical users
- Workarounds for known limitations}
~~~

### Internal: FAQ

**File:** `docs/generated/internal/faq.md`

~~~markdown
---
type: faq
audience: internal
status: draft
generated: {date}
source-tier: {tier}
hermes-version: 1.0.0
---

# Internal FAQ

{Organized by topic area. Each entry follows this format:}

## {Topic Area}

### Q: {Question — phrased as a customer or internal team member would ask it}
**A:** {Answer — direct, specific, actionable. Include:
- The factual answer
- Any caveats or exceptions
- Where to find more detail (link to feature handoff or use case doc)
- Escalation path if the answer doesn't resolve the issue}

**Related:** [{link to feature handoff}] [{link to use case doc}]
~~~

**Generation rules for FAQ:**
1. Mine questions from: feature handoff "Common Questions" sections, use case "CS Notes", troubleshooting guides
2. Add questions derived from feature limitations and edge cases
3. Group by topic area (one per feature/domain)
4. Minimum 3 questions per topic area
5. Every answer must be actionable — never "it depends" without specifics
6. Cross-link to source docs

### Internal: Troubleshooting Guide

**File:** `docs/generated/internal/troubleshooting/ts-{NNN}-{slug}.md`

~~~markdown
---
id: ts-{NNN}
type: troubleshooting
audience: internal
topic: {symptom-area}
status: draft
generated: {date}
source-tier: {tier}
hermes-version: 1.0.0
---

# Troubleshooting: {Symptom Area}

## Symptoms
{Observable symptoms that indicate this problem — what the user sees or reports:
- "{exact error message or behavior}"
- "{visible symptom in UI/API/CLI}"}

## Quick Check
{First thing to verify — the most common cause:
1. Check {X} — if {condition}, that's the issue. Fix: {resolution}.
2. If not, proceed to diagnostic steps.}

## Diagnostic Steps
{Decision tree for isolating the root cause:

### Step 1: {Check}
- **Run:** {command or action}
- **If {result A}:** → {conclusion or next step}
- **If {result B}:** → {conclusion or next step}

### Step 2: {Next check}
- **Run:** {command or action}
- **If {result A}:** → Root cause is {X}. See Resolution below.
- **If {result B}:** → Escalate to {team} with {required info}.}

## Resolutions
{For each identified root cause:
### {Root Cause}
- **Fix:** {exact steps to resolve}
- **Verify:** {how to confirm the fix worked}
- **Prevent:** {how to prevent recurrence, if applicable}}

## Escalation
{When to escalate and what info to provide:
- **Escalate to:** {team or person}
- **Include:** {required diagnostic data}
- **SLA:** {expected response time}}

## Related
{Links to relevant feature handoffs, use cases, and error references}
~~~

**Generation rules for troubleshooting:**
1. Derive symptoms from error handling code, known error messages, and exception types
2. Structure as decision trees — symptom → diagnosis → resolution
3. Always start with the most common cause (quick check)
4. Include exact commands and expected output at each diagnostic step
5. Every path must terminate in either a resolution or an escalation

### Internal: Glossary

**File:** `docs/generated/internal/glossary.md`

~~~markdown
---
type: glossary
audience: internal
status: draft
generated: {date}
source-tier: {tier}
hermes-version: 1.0.0
---

# Glossary

{Alphabetically ordered. Each term follows this format:}

### {Term}
**Definition:** {Clear, concise definition — 1-2 sentences}
**Context:** {Where this term is used — which features, APIs, or UI elements}
**Also known as:** {Aliases, abbreviations, or alternative names — if any}
**Not to be confused with:** {Similar terms that mean different things — if any}
~~~

**Generation rules for glossary:**
1. Extract terms from: code identifiers (models, types, enums), API parameter names, config keys, UI labels
2. Include domain-specific terms that non-engineers might not know
3. Include terms that are used differently in this project than their common meaning
4. Alphabetical order, no exceptions
5. Keep definitions under 2 sentences — link to feature docs for detail

### External: API Reference

**File:** `docs/generated/external/api-reference.md`

~~~markdown
---
type: api-reference
audience: external
status: draft
generated: {date}
source-tier: {tier}
hermes-version: 1.0.0
---

# API Reference

## Authentication
{How to authenticate — API keys, OAuth, JWT, etc. Include exact header format:
```
Authorization: Bearer {your-token}
```}

## Base URL
{Base URL for all endpoints:
```
https://api.{product}.com/v1
```}

## Endpoints

### {Resource Name}

#### {METHOD} {/path}
{1-sentence description of what this endpoint does}

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| ... | ... | ... | ... |

**Request Body:**
```json
{example request body}
```

**Response:**
```json
{example successful response}
```

**Status Codes:**
| Code | Meaning |
|------|---------|
| 200 | Success |
| 400 | Invalid request — {common causes} |
| 401 | Authentication required |
| 404 | Resource not found |

**Example:**
```bash
curl -X {METHOD} https://api.{product}.com/v1/{path} \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{request body}'
```
~~~

**Generation rules for API reference:**
1. If OpenAPI/Swagger spec exists: parse it for endpoints, params, schemas, responses
2. If no spec: scan route handlers, controller files, and API test files
3. Every endpoint needs: method, path, description, parameters, request/response examples, status codes
4. Include curl examples for every endpoint
5. Group endpoints by resource/domain
6. Document authentication once at the top, reference it from endpoints

### External: Feature Docs

**File:** `docs/generated/external/features/feat-{NNN}-{slug}.md`

~~~markdown
---
id: feat-{NNN}
type: feature-doc
audience: external
topic: {feature-name}
status: draft
generated: {date}
source-tier: {tier}
hermes-version: 1.0.0
---

# {Feature Name}

## Overview
{What this feature does — 2-3 sentences in user-facing language. Focus on the benefit, not the implementation.}

## How to Use It
{Step-by-step instructions for the most common use case:
1. {Step — with exact UI paths, commands, or API calls}
2. {Step}
3. {Step}}

## Configuration
{User-configurable options:
| Option | Description | Default |
|--------|-------------|---------|
| ... | ... | ... |}

## Examples
{2-3 practical examples showing different use cases:

### Example: {Use Case}
{Steps or code showing this specific use case}
}

## Limitations
{What this feature doesn't do or known constraints — be honest:
- {limitation}}

## Related
{Links to related features, API endpoints, guides}
~~~

### External: Getting Started

**File:** `docs/generated/external/getting-started.md`

~~~markdown
---
type: getting-started
audience: external
status: draft
generated: {date}
source-tier: {tier}
hermes-version: 1.0.0
---

# Getting Started

## Prerequisites
{What you need before starting:
- {requirement} (version {X} or higher)
- {requirement}}

## Installation
{Minimum steps to install — copy-pasteable commands:
```bash
{install command}
```}

## Quick Start
{The fastest path to "hello world" — 5 steps or fewer:

### 1. {First step}
```bash
{command}
```

### 2. {Next step}
{instruction}

### 3. Verify it works
```bash
{verification command}
```
Expected output:
```
{what you should see}
```}

## Next Steps
{Where to go from here:
- [{Feature X}](features/feat-001-x.md) — {why you'd use it}
- [API Reference](api-reference.md) — {for programmatic access}
- [Configuration](config-reference.md) — {to customize behavior}}
~~~

### External: Config Reference

**File:** `docs/generated/external/config-reference.md`

~~~markdown
---
type: config-reference
audience: external
status: draft
generated: {date}
source-tier: {tier}
hermes-version: 1.0.0
---

# Configuration Reference

## Configuration File
{Where the config lives and its format:
```
{path/to/config.{ext}}
```}

## Environment Variables
{Environment variables that affect behavior:
| Variable | Description | Default | Example |
|----------|-------------|---------|---------|
| ... | ... | ... | ... |}

## Options

### {Category}

#### `{option.key}`
- **Type:** {string|number|boolean|array|object}
- **Default:** `{default value}`
- **Required:** {yes|no}
- **Description:** {What this option controls}
- **Example:**
  ```yaml
  {option.key}: {example value}
  ```
- **Notes:** {Any caveats, interactions with other options, or version-specific behavior}
~~~

**Generation rules for config reference:**
1. Parse config schemas, type definitions, config parsing code, .env.example files
2. Every option needs: type, default, description, example
3. Group by category (not alphabetically)
4. Document interactions between options ("setting X requires Y to be true")
5. Include environment variable overrides if supported

### External: Error Reference

**File:** `docs/generated/external/error-reference.md`

~~~markdown
---
type: error-reference
audience: external
status: draft
generated: {date}
source-tier: {tier}
hermes-version: 1.0.0
---

# Error Reference

## Error Format
{How errors are returned — HTTP status codes, error response schema, error codes:
```json
{
  "error": {
    "code": "{ERROR_CODE}",
    "message": "{human-readable message}",
    "details": {}
  }
}
```}

## Errors

### {ERROR_CODE}
- **HTTP Status:** {status code}
- **Message:** "{typical error message}"
- **Cause:** {What triggers this error}
- **Resolution:** {How to fix it — specific, actionable steps}
- **Example:**
  ```json
  {example error response}
  ```
~~~

**Generation rules for error reference:**
1. Extract error codes, messages, and HTTP status codes from error handling code
2. Every error needs: code, status, message, cause, resolution
3. Group by category (auth errors, validation errors, server errors)
4. Resolution must be actionable from the user's perspective — not "fix the code" but "check your API key"

### External: Migration Guide

**File:** `docs/generated/external/migration/v{from}-to-v{to}.md`

~~~markdown
---
type: migration-guide
audience: external
from-version: {from}
to-version: {to}
status: draft
generated: {date}
source-tier: {tier}
hermes-version: 1.0.0
---

# Migration Guide: v{from} → v{to}

## Overview
{Summary of what changed and why migration is needed — 2-3 sentences}

## Breaking Changes

### {Change Title}
**What changed:** {description of the breaking change}
**Why:** {brief rationale}
**Before (v{from}):**
```{lang}
{old code/config/API call}
```
**After (v{to}):**
```{lang}
{new code/config/API call}
```
**Migration steps:**
1. {step}
2. {step}

## Deprecated Features
{Features deprecated in this version — still work but will be removed:
| Feature | Replacement | Removal Target |
|---------|-------------|---------------|
| ... | ... | v{X} |}

## New Features
{Features added that may affect your workflow:
- **{Feature}:** {brief description} — [docs](../features/feat-XXX.md)}

## Checklist
{Ordered checklist for migration:
- [ ] {step 1}
- [ ] {step 2}
- [ ] Verify: {how to confirm migration succeeded}}
~~~

**Generation rules for migration guide:**
1. Parse git log between version tags for breaking changes
2. Detect API signature changes, removed endpoints, renamed config keys
3. Every breaking change needs before/after examples
4. Include a migration checklist at the end
5. Only generated when a version change introduces breaking changes

### Marketing: Release Announcement

**File:** `docs/generated/marketing/release-announcements/ra-v{version}.md`

~~~markdown
---
id: ra-v{version}
type: release-announcement
audience: marketing
version: {version}
status: draft
generated: {date}
source-tier: {tier}
hermes-version: 1.0.0
---

# {Product Name} v{version}: {Headline}

## The Headline
{One compelling sentence — what's the most exciting thing in this release. Usable as email subject, social post, or blog title.}

## What's New
{For each notable feature/improvement — user-benefit language:

### {Feature Name}
{2-3 sentences: what it enables, who benefits, why it matters. No technical jargon.}

**Key benefit:** {single sentence — the "so what?"}
}

## Improvements
{Bulleted list of quality-of-life improvements — brief:
- {improvement}: {benefit}}

## Getting Started
{How to access the new features — upgrade instructions, links to docs:
- Existing users: {upgrade path}
- New users: {getting started link}}

## What's Next
{Brief preview of upcoming work — builds anticipation without committing:
"We're working on {X} and {Y} — stay tuned."}
~~~

### Marketing: Sales One-Pager

**File:** `docs/generated/marketing/one-pagers/op-{NNN}-{slug}.md`

~~~markdown
---
id: op-{NNN}
type: one-pager
audience: marketing
topic: {feature-or-product}
status: draft
generated: {date}
source-tier: {tier}
hermes-version: 1.0.0
---

# {Feature/Product Name}

## The Problem
{2-3 sentences describing the pain point — in the prospect's language}

## The Solution
{2-3 sentences describing how {product/feature} solves it — benefit-focused}

## Key Benefits
{3-4 benefits with proof points:
- **{Benefit}:** {supporting detail or metric}}

## How It Works
{Simplified 3-step explanation:
1. **{Step}** — {what happens}
2. **{Step}** — {what happens}
3. **{Step}** — {result}}

## Who It's For
{Target personas:
- **{Role}:** {why they care}
- **{Role}:** {why they care}}

## Get Started
{Clear CTA — what to do next}
~~~

**Generation rules for sales one-pager:**
1. Derive from the corresponding feature brief — distill to essentials
2. Must fit on one printed page (~400 words max)
3. Problem → Solution → Benefits → How → Who → CTA flow
4. Every sentence must pass the "so what?" test
5. No technical jargon — a VP should understand every word

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
- Hermes version: 1.0.0

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
mkdir -p docs/generated/internal/use-cases
mkdir -p docs/generated/internal/troubleshooting
mkdir -p docs/generated/external/features
mkdir -p docs/generated/external/migration
mkdir -p docs/generated/marketing/feature-briefs
mkdir -p docs/generated/marketing/release-announcements
mkdir -p docs/generated/marketing/one-pagers
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

**Internal — Use Case Docs:**
1. For each distinct user scenario within this feature, generate a use case doc
2. Focus on user workflows, not code modules — one doc per scenario, not per function
3. Include the full flow: trigger → steps → variations → edge cases → data impact
4. The "CS Notes" section bridges the gap between technical behavior and customer communication
5. Write files to `docs/generated/internal/use-cases/uc-{NNN}-{slug}.md`

**Internal — Troubleshooting Guide:**
1. Identify error scenarios from error handling code, exception types, and known failure modes
2. Structure as decision trees: symptom → quick check → diagnostic steps → resolution
3. Always start with the most common cause
4. Every diagnostic step must have a concrete command or action with expected output
5. Write files to `docs/generated/internal/troubleshooting/ts-{NNN}-{slug}.md`

**External — Feature Docs:**
1. Generate user-facing documentation for each feature
2. Voice: clear, second-person ("You can...", "To configure X, do Y")
3. Focus on what the user can DO, not how it works internally
4. Include practical examples — 2-3 per feature
5. Be honest about limitations
6. Write files to `docs/generated/external/features/feat-{NNN}-{slug}.md`

**Marketing — Feature Brief:**
1. Gather feature context + any business context from context files
2. Generate the feature brief using the template above
3. Voice: value-driven, benefit-first. Write for a product marketer.
4. Translate every technical capability into a user benefit
5. The "Suggested Messaging" section should be ready-to-use copy
6. Write file to `docs/generated/marketing/feature-briefs/fb-{NNN}-{slug}.md`

**Marketing — Sales One-Pager:**
1. Derive from the corresponding feature brief — distill to essentials
2. Must be under ~400 words (fits on one printed page)
3. Follow Problem → Solution → Benefits → How → Who → CTA flow
4. No technical jargon — a VP should understand every word
5. Write file to `docs/generated/marketing/one-pagers/op-{NNN}-{slug}.md`

**3c. Generate cross-feature docs (not per-feature, but per-project):**

**Internal Release Notes** (if version context is available):
1. Parse git log for the relevant commit range
2. Categorize commits by type and affected feature
3. Generate internal release notes using the template
4. Write to `docs/generated/internal/release-notes/rn-v{version}.md`

**Internal FAQ:**
1. Aggregate common questions from all feature handoffs
2. Add questions derived from edge cases, limitations, and troubleshooting guides
3. Group by topic area, minimum 3 Q&A per area
4. Cross-link to source docs
5. Write to `docs/generated/internal/faq.md`

**Internal Glossary:**
1. Extract terms from code identifiers, API parameters, config keys, UI labels
2. Include domain-specific terms and terms used differently than their common meaning
3. Alphabetical order
4. Write to `docs/generated/internal/glossary.md`

**External Changelog:**
1. Parse git log
2. Translate to user-facing language
3. Follow Keep a Changelog format
4. Write to or append `docs/generated/external/changelog.md`

**External API Reference** (if API surface exists):
1. Parse OpenAPI/Swagger spec if available, otherwise scan route handlers and controllers
2. Document every endpoint: method, path, params, request/response, status codes, curl examples
3. Document authentication at the top
4. Write to `docs/generated/external/api-reference.md`

**External Getting Started:**
1. Identify the minimum path to a working "hello world"
2. Include prerequisites, installation, quick start (5 steps or fewer), verification
3. Link to feature docs and API reference for next steps
4. Write to `docs/generated/external/getting-started.md`

**External Config Reference:**
1. Parse config schemas, type definitions, .env.example files
2. Document every option: type, default, description, example
3. Group by category
4. Write to `docs/generated/external/config-reference.md`

**External Error Reference:**
1. Extract error codes and messages from error handling code
2. Every error needs: code, status, message, cause, resolution
3. Resolution must be actionable from the user's perspective
4. Write to `docs/generated/external/error-reference.md`

**External Migration Guide** (only when breaking changes exist between versions):
1. Detect breaking changes from git log between version tags
2. Generate before/after examples for each breaking change
3. Include migration checklist
4. Write to `docs/generated/external/migration/v{from}-to-v{to}.md`

**Marketing Product Datasheet:**
1. Aggregate all features, specs, and capabilities
2. Extract technical specs from manifests, configs, Dockerfiles
3. Generate datasheet using the template
4. Write to `docs/generated/marketing/datasheet.md`

**Marketing Release Announcement** (if version context is available):
1. Derive from internal release notes — reframe for external value
2. Lead with the most exciting feature — write a compelling headline
3. User-benefit language, no technical jargon
4. Include upgrade path and "what's next" preview
5. Write to `docs/generated/marketing/release-announcements/ra-v{version}.md`

**3d. After each audience is complete:**
- Update `.hermes.md` — mark audience as `done`, update timestamp
- If single-audience mode: stop here

### Step 4: Generate Index

Write `docs/generated/index.md`:

```markdown
# Hermes Documentation Index

Generated: {timestamp} | Source: {tier} | Hermes v1.0.0

## Internal ({count} docs)
| ID | Type | Topic | Generated |
|----|------|-------|-----------|
| fh-001 | Feature Handoff | {topic} | {date} |
| uc-001 | Use Case | {topic} | {date} |
| ts-001 | Troubleshooting | {topic} | {date} |
| rn-v{ver} | Release Notes | v{ver} | {date} |
| faq | FAQ | All | {date} |
| glossary | Glossary | All | {date} |

## External ({count} docs)
| ID | Type | Topic | Generated |
|----|------|-------|-----------|
| getting-started | Getting Started | All | {date} |
| feat-001 | Feature Doc | {topic} | {date} |
| api-ref | API Reference | All | {date} |
| config-ref | Config Reference | All | {date} |
| error-ref | Error Reference | All | {date} |
| changelog | Changelog | All | {date} |
| migration | Migration Guide | v{X}→v{Y} | {date} |

## Marketing ({count} docs)
| ID | Type | Topic | Generated |
|----|------|-------|-----------|
| fb-001 | Feature Brief | {topic} | {date} |
| op-001 | One-Pager | {topic} | {date} |
| ds-001 | Datasheet | Full Product | {date} |
| ra-v{ver} | Release Announcement | v{ver} | {date} |
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
- **Internal:** Generate or update use case docs for changed scenarios.
- **Internal:** Update troubleshooting guides if error handling changed.
- **Internal:** Generate new release notes for the new version/commit range.
- **Internal:** Update FAQ with any new common questions from updated handoffs.
- **Internal:** Update glossary if new terms were introduced.
- **External:** Generate or update feature docs for changed features.
- **External:** Append new entries to changelog.
- **External:** Update API reference if endpoints changed.
- **External:** Update config reference if config options changed.
- **External:** Update error reference if error codes changed.
- **External:** Generate migration guide if breaking changes exist.
- **External:** Update getting started if setup process changed.
- **Marketing:** Generate new feature brief only for NEW features. Update existing briefs for modified features.
- **Marketing:** Generate sales one-pager for significant new features.
- **Marketing:** Generate release announcement for the new version.

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

**All docs:**
1. **Frontmatter is complete** — all required fields populated
2. **No placeholder text** — no `{TODO}`, `TBD`, or `[fill in]` remaining
3. **Voice matches audience** — internal docs aren't marketing-speak, marketing docs aren't technical jargon
4. **Cross-references are valid** — links to other docs point to files that exist

**Internal docs:**
5. **Feature Handoff has Common Questions** — never empty, minimum 3 Q&A pairs
6. **Use Case docs have complete flows** — every step has both user action and system response
7. **Troubleshooting has decision paths** — every branch terminates in resolution or escalation
8. **FAQ answers are actionable** — never "it depends" without specifics
9. **Glossary terms are alphabetical** — no exceptions

**External docs:**
10. **Changelog entries are user-facing** — no internal refactors or CI changes unless they affect users
11. **API Reference has curl examples** — every endpoint has a working example
12. **Getting Started works in under 5 steps** — if it takes more, simplify
13. **Config Reference has defaults** — every option shows its default value
14. **Error Reference has resolutions** — every error has actionable fix steps
15. **Migration Guide has before/after** — every breaking change shows old and new code

**Marketing docs:**
16. **Feature Briefs have Suggested Messaging** — never empty, minimum 2 messages
17. **Datasheet specs are factual** — only include what's verifiable from code/config, not aspirational
18. **One-Pagers are under 400 words** — must fit on one printed page
19. **Release Announcements lead with value** — headline is benefit-focused, not feature-named
20. **No technical jargon in marketing docs** — a VP should understand every word
