# Marketing Document Templates

Templates for product marketing, sales, and content teams.

**Voice:** Confident, benefit-focused, concise. Product-centric ("Now you can...", "{Product} enables..."). Reader cares about outcomes, not mechanisms. Value over detail — every sentence should answer "so what?"

---

## Feature Brief (Keystone Document)

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

---

## Product Datasheet

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

---

## Release Announcement

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

---

## Sales One-Pager

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
