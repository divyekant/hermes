# External Document Templates

Templates for end users, developers, and integrators.

**Voice:** Clear, helpful, encouraging. Second person ("You can...", "To do X, run Y"). Reader may be new to the project. Don't assume knowledge of internals. Clarity over completeness — lead with the most common use case.

---

## Changelog

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

---

## API Reference

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

---

## Feature Docs

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

---

## Getting Started

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

---

## Config Reference

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

---

## Error Reference

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

---

## Migration Guide

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
