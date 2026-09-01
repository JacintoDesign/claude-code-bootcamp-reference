# Module 1 Reference Guide
### Claude — Project Workflow: Prompts, Documents, and Director's Checklists

Use this guide alongside the Module 1 videos.

---

## Table of Contents

1. [Demo: Start with your Browser and an Idea...](#demo-start-with-your-browser-and-an-idea)
2. [The Three Standards](#the-three-standards)
3. [Web App Anatomy](#web-app-anatomy)
4. [Product Requirement Document (PRD)](#product-requirement-document-prd)
5. [Content Model](#content-model)
6. [Site Contract](#site-contract)
7. [Design Standard Document](#design-standard-document)
8. [Programming Agents](#programming-agents)
9. [Claude Project](#claude-project)
10. [Building a Website](#building-a-website)
11. [The Director's Audit](#the-directors-audit)
12. [Finishing the Director's Audit](#finishing-the-directors-audit)

---

## Core Concepts

**Creative direction becomes engineering when it is encoded.** A vibe sentence can produce a compelling first result. The PRD, content model, site contract, design standard, and `CLAUDE.md` make that direction repeatable.

**The three standards apply to the whole pipeline.** Ask whether the work scales, whether it is secure, and whether you understand why it works.

**Planning documents have separate jobs.** The PRD defines the product and success; the content model defines what exists; the site contract defines structure and behaviour; the design standard defines the visual system; `CLAUDE.md` points the agent to the right source of truth.

**Context is a resource.** Keep `CLAUDE.md` brief and load supporting documents when the task requires them.

**Build atomically.** One meaningful unit is built, checked against the contract and design standard, understood, and accepted before the next unit starts.

**The Director's Audit is comparative.** Judge the artefact against its written specification, not against whether it merely looks plausible or runs.

---

## Demo: Start with your Browser and an Idea...

### Vibe sentence examples

- _3am in Tokyo. You've just missed the last train. Neon reflections on wet pavement._
- _A sun-bleached roadside diner in New Mexico, 1987._
- _The inside of a greenhouse in early morning fog._
- _Scandinavian winter. Minimal. A little melancholy. Very clean._

### Browser build prompt

> _"Turn this into a complete, styled HTML and CSS landing page UI. Use this feeling to define every design decision — the colour palette, typography, spacing, texture, and layout. The page should feel like you walked into that world. Return only the complete HTML file with CSS embedded."_

### Student workflow

1. Open Claude in the browser and enter your own one-sentence world or atmosphere.
2. Add the build prompt above.
3. Copy the result into CodePen and render it.
4. Run the same sentence and prompt in a fresh conversation to see how a direction can produce different valid executions.
5. Save your vibe sentence, the generated HTML, and a screenshot. The sentence follows the project through the whole module.

---

## The Three Standards

Use these questions to evaluate every plan, implementation, and release:

### 1. Does it scale?

- Can the structure accept another section, screen, or feature without a rewrite?
- Are repeated values and behaviours centralized rather than copied?
- Could another agent continue the work from the documents and current state?

### 2. Is it secure?

- Does the build stay inside the declared technical scope?
- Are secrets, credentials, and sensitive data absent from source and prompts?
- Were unapproved dependencies, external scripts, or data flows introduced?

### 3. Do I know exactly why it works?

- Can you explain the page structure and key behaviours?
- Can you trace a decision to a requirement or design rule?
- Can you identify where you would make a future change?

These standards apply across the full workflow:

```text
Idea → Specification → Agent direction → Implementation → Verification → Release
```

---

## Web App Anatomy

### The three layers

| Layer | Responsibility | Typical examples |
|---|---|---|
| Frontend | What the user sees and interacts with | HTML, CSS, React, forms, navigation |
| Backend | Server-side logic and integrations | API routes, authentication, business rules |
| Database | Persistent structured data | users, messages, projects, relationships |

### Data flow to understand

```text
User action → Frontend request → Backend validation/logic → Database or external API
           ← Frontend render  ← Structured response       ← Result
```

For this module's site, the declared constraint is important: it is a static, single-page frontend with no backend, database, or authentication. Anything requiring those layers belongs outside the Must-Have scope.

### Director's review

- Which layer owns each responsibility?
- Where does data originate, and where does it end?
- Where can failure occur?
- Has the agent added a layer the project does not need?

---

## Product Requirement Document (PRD)

The PRD defines what the product is, who it serves, what it must do, its constraints, and what counts as success.

### PRD generation prompt

> _"I'm building a single-page static website. Here is the atmosphere I want it to embody: [your vibe sentence]._
>
> _Generate a Product Requirements Document with these six sections: Vision, User Personas, Feature List (Must-Have / Should-Have / Nice-to-Have), Technical Constraints, Design Standard summary, and Success Criteria._
>
> _Important context: this is a static site — no backend, no database, no authentication. The Must-Have feature list should reflect that constraint. The Design Standard section should be a brief summary — two to four sentences capturing the visual atmosphere. Keep the whole document concise — every sentence should be necessary."_

### Required sections

1. Vision
2. User Personas
3. Feature List — Must-Have / Should-Have / Nice-to-Have
4. Technical Constraints
5. Design Standard summary
6. Success Criteria

### Director's review checklist

- Does the vision match the product you actually want?
- Are personas concrete people with situations and needs rather than demographics?
- Is the Must-Have list honest about the static-site scope?
- Is the design summary a useful orientation rather than a duplicate specification?
- Is every success criterion measurable and manually checkable?

Save the approved result as `PRD.md`.

---

## Content Model

The content model lists every page section in order and every field inside each section. It prevents the agent from inventing structure or filler content.

### Content model prompt

> _"Here is my PRD: [paste PRD]_
>
> _Based on this, map out a content model for the site. List each section of the page in order with the specific content fields each section contains. For each field, add a brief description and any constraints — length, format, tone. Keep it minimal — only what's essential for the Must-Have features. Flag any gaps between the PRD and the content model."_

### Field pattern

```markdown
## Hero
- Headline — eight words maximum; carries the emotional core
- Subheadline — one sentence; expands without repeating
- Primary CTA — action-oriented label; links to the intended section
```

### Director's review checklist

- Does every Must-Have feature appear in the content model?
- Is the page order intentional?
- Does every field have a useful length, format, or tone constraint?
- Did the agent add any section that the PRD does not justify?

Save the approved result as `content-model.md`.

---

## Site Contract

The content model defines what exists. The site contract defines the page structure and how every important interaction behaves.

### Site contract prompt

> _"Based on my PRD and content model, write a site contract for my single-page website._
>
> _The contract has three sections:_
>
> _Page Structure — every section in page order, with the purpose of each section stated in one sentence._
>
> _Content Inventory — for each section, every content field with a brief description and any constraints on length, format, or tone._
>
> _Behaviour Spec — navigation behaviour, scroll behaviour, hover states by element type, mobile layout changes, and any transitions or animations._
>
> _This is a static site — no forms submitting to a backend, no authentication, no database. All behaviour is CSS and JavaScript only._
>
> _Here is the PRD: [paste PRD] Here is the content model: [paste content model]"_

### Contract entry pattern

```markdown
## Hero
Purpose: Establish the atmosphere and communicate the core proposition.

Content:
- headline — string; eight words maximum
- subheadline — string; one sentence
- primaryCta — label and section target

Behaviour:
- CTA scrolls to the named section
- Layout stacks at the mobile breakpoint
- Entrance motion respects prefers-reduced-motion
```

### Director's review checklist

- Does page order match the content model?
- Does every section have a clear purpose?
- Are all navigation, scroll, hover, mobile, and transition behaviours explicit?
- Are static-site boundaries preserved?
- Could two developers implement the same behaviour from this contract?

Save the approved result as `site-contract.md`.

---

## Design Standard Document

The design standard translates the atmosphere into visual decisions specific enough to reproduce consistently.

### Design standard prompt

> _"I'm building a single-page website. Here is the atmosphere I want it to embody: [your vibe sentence]._
>
> _Write a design standard document that translates this atmosphere into a complete visual specification. Cover:_
>
> _Colour palette — background colours, text colours, accent colours, with approximate hex values and a note on how each is used._
>
> _Typography — font families, weight choices, size relationships between heading levels and body text, line height, letter spacing._
>
> _Spacing — the philosophy expressed as concrete values. What's the base unit? What are the standard padding values?_
>
> _Texture and atmosphere — any grain, gradient, blur, or depth effects that carry the world._
>
> _Interactions — transition timing, easing, hover state behaviour, the emotional quality of movement._
>
> _Write this as a reference document a developer would consult when making any visual decision. Be specific enough that two developers reading it would make the same choices."_

### Required decisions

- Named colour tokens with approximate hex values and usage rules
- Heading and body font families, weights, scale, line height, and tracking
- Base spacing unit and standard section/container values
- Texture, atmosphere, depth, and restraint rules
- Motion durations, easing, hover behaviour, and reduced-motion treatment
- Accessibility floor for contrast, focus, and legibility

Save the approved result as `design-standard.md`.

---

## Programming Agents

`CLAUDE.md` is the short standing-order file. It identifies the project, states hard constraints, and points to supporting documents. It should not duplicate them.

### CLAUDE.md generation prompt

> _"Write a CLAUDE.md file for the following project. It should be brief — under one page. It contains three things only: a two to three sentence project identity statement, the technical stack with explicit constraints, and a list of supporting documents with a one-line description of each and when to read it._
>
> _Project identity: [paste your vibe sentence and the Vision from your PRD]_
>
> _Stack: static single-page site, one HTML file, embedded CSS and JavaScript, no frameworks, no npm, no build tools._
>
> _Supporting documents: design-standard.md, site-contract.md, PRD.md, content-model.md._
>
> _Do not include any detailed content that belongs in the supporting documents. If information is already in one of the supporting documents, the CLAUDE.md should point to that document rather than repeat it."_

### Expected document set

```text
CLAUDE.md
PRD.md
content-model.md
site-contract.md
design-standard.md
```

### Atomic execution pattern

```text
Read the relevant documents
→ Build one meaningful unit
→ Check it against the contract
→ Check it against the design standard
→ Confirm you understand it
→ Accept it as the new baseline
```

Build navigation and page structure first, hero second, content sections in contract order, and footer last. Start each new build conversation with a screenshot of the current artefact to reduce visual drift.

---

## Claude Project

### Project knowledge base

Create a Claude Project and upload:

- `CLAUDE.md`
- `design-standard.md`
- `site-contract.md`
- `PRD.md`
- `content-model.md`

Open each uploaded document once and confirm it is complete rather than truncated.

### Prompting pattern

Every precise build prompt should identify:

1. Which documents to read
2. The exact deliverable
3. The boundary — what not to build or change
4. What verified looks like

Before implementation, you can surface the agent's intended decisions:

> _"Before you build anything, tell me which design decisions you're planning to make and which documents you're deriving them from."_

### Knowledge-base verification prompts

> _"Read CLAUDE.md and summarise: what is this project, what are the technical constraints, and what supporting documents are available?"_

> _"Read the design standard and tell me: what is the background colour for this project and what is the primary accent colour?"_

If either answer is generic or inaccurate, verify the uploads and the pointers in `CLAUDE.md` before starting the build.

### Targeted correction pattern

```text
[Specific element] is [observable problem] rather than [specified result].
Update [specific token/property/behaviour] to match [source document].
Do not change [working layout/content/behaviour].
```

Make one correction at a time and inspect the artefact after each change.

---

## Building a Website

### Build sequence

1. Navigation and overall page structure
2. Hero
3. Remaining content sections in site-contract order
4. Footer
5. Whole-site refinement pass

### Navigation and structure prompt

> _"Read CLAUDE.md, the site contract, and the design standard. Build the navigation and overall page structure only — the sticky header, the page container layout, and the section spacing system. Do not build any content sections yet. Follow the navigation behaviour specified in the site contract exactly. Apply the colour palette and typography from the design standard. This is complete when the navigation renders correctly, transitions from transparent to solid at the specified scroll depth, and the page container layout is in place for the content sections that follow."_

### Hero prompt

> _"Here is the current state of the site: [screenshot]. Read the site contract and the design standard. Build the hero section only, following the content inventory exactly — [your headline, subheadline, and CTA as specified]. Apply the full design system from the design standard. The hero should establish the atmospheric tone of the entire site. This is complete when the content fields are present as specified, the visual treatment matches the design standard, and the section feels like the world described in the project identity."_

### Content-section prompt template

> _"Here is the current state of the site: [screenshot]. Read CLAUDE.md, the site contract, and the design standard. Build the [section name] section only. Use the exact content fields and behaviour specified for that section. Match the established visual treatment in the screenshot without overriding the design standard. Do not modify completed sections. This is complete when [section-specific verification criteria]."_

### Before accepting each unit

- The section matches its purpose, fields, and behaviour in `site-contract.md`.
- Colour, typography, spacing, texture, and motion match `design-standard.md`.
- The new section feels continuous with the current artefact.
- You understand the structure and know where future changes belong.
- No previously accepted section changed unexpectedly.

Copy the final artefact link, save the full HTML, and take a screenshot for the audit.

---

## The Director's Audit

Open the rendered site, `site-contract.md`, and `design-standard.md` together. Record findings first; do not start fixing until the audit is complete.

### Dimension 1 — Structural integrity

- Every contracted section is present and in the correct order.
- Every section serves its declared purpose.
- Every content field is present and respects its constraints.
- Every specified behaviour can be demonstrated.

### Dimension 2 — Consistency

- Colours are centralized as CSS custom properties.
- No unapproved inline styles or dependencies were introduced.
- JavaScript follows the stated project conventions.
- Every hard constraint in `CLAUDE.md` was followed.

### Dimension 3 — Visual quality

Evaluate colour, typography, spacing, atmosphere, texture, hover states, and motion against the design standard and the original vibe sentence.

### Multimodal visual audit prompt

Attach a screenshot in a fresh conversation outside the Project, then use:

> _"I am auditing the visual quality of this website against a design standard. Here is the design standard:_
>
> _[paste the full contents of your design-standard.md]_
>
> _Evaluate the site against the design standard across five dimensions: colour palette, typography, spacing and layout, atmosphere and texture, and interaction quality. Note that interaction states are not visible in the screenshot — flag that dimension for manual verification._
>
> _For each dimension: state specifically where the site matches the standard and where it deviates. Be precise — not 'the colours feel slightly off' but 'the background reads as cool grey rather than the specified deep navy, which creates a clinical rather than intimate atmosphere.' If a dimension fully meets the standard, say so briefly and move on."_

### Dimension 4 — Reviewer Agent

Open another fresh conversation and provide the full source plus the three governing documents:

> _"You are conducting a professional code review. Your role is to find problems — not to validate or reassure. Be rigorous, be specific, and be direct._
>
> _Review the following HTML, CSS, and JavaScript against the specification documents provided. For each issue you find: the category (Structural / Consistency / Visual), the exact location in the code, a clear description of the problem, and a specific suggested resolution._
>
> _If a dimension is clean, state that explicitly and move on._
>
> _CLAUDE.md: [paste] design-standard.md: [paste] site-contract.md: [paste]_
>
> _Site code: [paste full HTML]"_

Treat agent findings as evidence to inspect. Agreement between your audit and an independent reviewer is high-confidence; disagreement requires investigation and a reasoned decision.

---

## Finishing the Director's Audit

### Fix order

1. Structural gaps
2. Consistency violations
3. Visual-quality issues

Fix one finding per message, inspect the artefact, and confirm that no accepted work regressed.

### Audit log template

```markdown
# Director's Audit Log

## Vibe Site — [date]
Auditor: Director + Reviewer Agent

### Structural
[Finding and resolution — or: Clean]

### Consistency
[Finding and resolution — or: Clean]

### Visual Quality
[Finding and resolution — or: Clean]

### Reviewer Agent Additional Findings
[Anything caught that was not in your own review]

### Status
Verified. Site meets design standard and site contract.
```

Save this as `audit-log.md` and add it to the Claude Project knowledge base.

### Final acceptance checklist

- All priority findings are resolved and documented.
- The site still matches the original vibe sentence as a complete experience.
- The final source is saved outside the temporary artefact.
- The shareable artefact link opens successfully.
- The Project now contains the five planning documents plus `audit-log.md`.

---

## Complete Student Document Set

```text
CLAUDE.md
PRD.md
content-model.md
site-contract.md
design-standard.md
audit-log.md
```
