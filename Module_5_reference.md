# Module 5 Reference Guide
### Claude Design / Code — VibeMail Project UI: Prompts, States, and Handoff

Use this guide alongside the Module 5 videos.

---

## Table of Contents

1. [What is Claude Design?](#what-is-claude-design)
2. [Opening Up the Claude Design Tool](#opening-up-the-claude-design-tool)
3. [What Is a Behavioural Spec?](#what-is-a-behavioural-spec)
4. [Populating Our DESIGN_SPEC.md](#populating-our-design_specmd)
5. [Building the DESIGN.md System](#building-the-designmd-system)
6. [Setting Up the Claude Design System](#setting-up-the-claude-design-system)
7. [Iterating Our Email UI in Claude Design](#iterating-our-email-ui-in-claude-design)
8. [Auditing Our Claude Design](#auditing-our-claude-design)
9. [Producing a Migration Plan in Claude Code](#producing-a-migration-plan-in-claude-code)
10. [Migrating the Handoff Components into a Next.js Structure](#migrating-the-handoff-components-into-a-nextjs-structure)
11. [Visual Inspection with Our Agent and Accessibility Review](#visual-inspection-with-our-agent-and-accessibility-review)
12. [API Integration with UI](#api-integration-with-ui)
13. [Email Bodies & Attachments](#email-bodies--attachments)
14. [Final Pass & Deployment](#final-pass--deployment)

---

## Core Concepts

**Claude Design and Claude Code solve different problems.** Design is a fast visual decision surface. Code turns the approved handoff into production architecture, integration, accessibility, and deployment.

**Behaviour comes before appearance.** `DESIGN_SPEC.md` defines views, states, actions, errors, and mobile behaviour using exact contract fields and endpoints.

**The spec is a compass; `DESIGN.md` is the map.** The behavioural spec says what the interface does. The design-system document supplies exact visual tokens and hierarchy.

**Native design-system fields constrain future generations.** Uploaded reference documents inform the work; native tokens actively reduce drift.

**Sample data comes before live API data.** Verify the migrated component architecture and visual system before introducing authentication and endpoint uncertainty.

**The handoff is a specification and component source, not the final application architecture.** Preserve useful React components and CSS tokens while adapting routing, types, persistence, and API wiring deliberately.

---

## What is Claude Design?

### Use Claude Design for

- Generating and comparing visual directions quickly
- Iterating structure through chat
- Adjusting token values through the Tweaks Panel
- Making surgical visual changes with inline comments
- Producing responsive React component prototypes

### Use Claude Code for

- Next.js application architecture
- TypeScript conversion and shared component boundaries
- Authentication, API integration, persistence, and tests
- Accessibility verification and production deployment

### Three iteration channels

1. Chat — structural or behavioural change
2. Tweaks Panel — token-level values such as colour, radius, spacing, opacity
3. Inline comments — one specific element or local correction

Specify React when you need the full component-aware design and Tweaks Panel workflow.

---

## Opening Up the Claude Design Tool

### Throwaway first-generation prompt

> "Build a glassmorphic email inbox card component as a React component — a single message row on a deep navy background. Frosted glass card, sender name, subject, snippet, timestamp, a star icon. Unread state has a filled accent dot and slightly more opaque glass. Make it dark and sleek."

### Structural refinement prompt

> "The glass card needs to feel more translucent — increase the backdrop blur and reduce the fill opacity so the background shows through clearly."

### Inline/surgical refinement

> "The outlined star needs to be more visible on hover — feels like it disappears."

Use the examples gallery to learn interaction patterns, then discard the throwaway project. The production design begins only after the behavioural spec and design system are ready.

---

## What Is a Behavioural Spec?

A behavioural spec describes every meaningful state without prescribing implementation. For each view, define:

- Default state
- User actions
- Loading state
- Error states using exact contract codes
- Empty state where applicable
- Named modes or activation conditions
- Mobile and keyboard behaviour

### VibeMail views

| View | Primary contract |
|---|---|
| Inbox | `GET /api/v1/messages`; message patch routes |
| Thread | `GET /api/v1/threads/:threadId` |
| Compose | send and draft endpoints |
| Label filter | Message status and starred state |
| Search | `GET /api/v1/messages/search` |

Design away preventable errors. For example, do not call search until one character exists, so `MISSING_QUERY` never appears as a user-visible state.

---

## Populating Our DESIGN_SPEC.md

### Behavioural-spec generation prompt

> You are helping build a behavioural spec for a glassmorphic email frontend called VibeMail Glass. The attached CONTRACT.md defines the complete backend API.
>
> For each frontend view listed below, write a section in plain language covering the five core areas — default state, user actions, loading state, error states, empty state — plus any additional sections the view requires (named modes for compose, an activation condition for search, a label filter bar sub-section for inbox). Omit areas that cannot apply to a view (thread view has no empty state).
>
> Name every field using the exact field names from the Message model in CONTRACT.md §3. Name every error state using the exact error.code values from CONTRACT.md §4. Name every endpoint using the exact path from CONTRACT.md §4. Never invent field names, error codes, or endpoints.
>
> The five views and their primary endpoints:
>
> - Inbox — GET /api/v1/messages (§4.2) plus PATCH /api/v1/messages/:id (§4.4, §4.5)
> - Thread view — GET /api/v1/threads/:threadId (§4.6)
> - Compose drawer — POST /api/v1/messages (§4.3), POST /api/v1/drafts (§4.8), PATCH /api/v1/drafts/:id (§4.9), DELETE /api/v1/drafts/:id (§4.10)
> - Label filter — client-side filter on status and isStarred fields, no dedicated endpoint
> - Search — GET /api/v1/messages/search (§4.7)
>
> Additional requirements:
>
> - The inbox label filter bar always renders four fixed chips: All, Starred, Archive, Trash. Custom Gmail labels appear after.
> - Compose has three named states: default, reply mode, and draft-restored state. Specify each separately.
> - draftId is server-managed — flag it as never client-supplied.
> - status is derived from labelIds at write time — flag it as never set directly by the frontend.
> - The mobile keyboard behaviour for the compose drawer must be specified: maximum drawer height 58% of viewport.
>
> After the five view sections, add a Visual Direction section: three sentences describing heavy glassmorphism on a deep navy background, with opacity and blur specified as ranges rather than single values.
>
> Output only the spec document. No preamble, no explanation, no markdown wrapper.

### Create and commit

```bash
mkdir -p docs
touch docs/DESIGN_SPEC.md
git add docs/DESIGN_SPEC.md
git commit -m "docs: VibeMail Glass behavioural spec"
```

### Contract-sensitive rules

- `draftId` is server-managed and never supplied as a client field.
- `status` is derived from `labelIds`; the frontend never writes it directly.
- Inbox and search results use the same Message row shape.
- Compose has default, reply, and draft-restored modes.
- Drawer height is capped at 58% on keyboard-constrained mobile layouts.
- Error treatments are specific: field-level, inline banner, redirect, or silent indicator as the spec states.

Audit generated names against `CONTRACT.md`; do not accept paraphrased fields, codes, or paths.

---

## Building the DESIGN.md System

Choose a reference system for structural fit: dark palette, dense information, card hierarchy, and technical typography. The course adapts OpenCode.

```bash
mkdir -p docs/design-system
# Save the downloaded reference as docs/design-system/opencode.DESIGN.md
```

### Adaptation prompt

> "Read the file at docs/design-system/opencode.DESIGN.md. Produce a modified version at docs/design-system/VIBEMAIL_DESIGN.md with the following changes:
>
> 1. Replace all surface treatment values with glassmorphic equivalents: translucent backgrounds at 8–12% white opacity, backdrop-filter blur between 20–40px depending on elevation (higher elevation = stronger blur), thin borders at 15–20% white opacity, soft drop shadows for depth. These values override any opaque surface specifications in the original.
>
> 2. Replace the primary typeface with JetBrains Mono. Use the variable font (JetBrainsMono[wght].ttf) available from Google Fonts. Keep all type scale values from the original — only the font family changes.
>
> 3. Adjust background colour values to: base background #050810, elevated background #0a0f1e. Preserve all other colour tokens from the original.
>
> 4. Add a section at the top: 'Adapted from OpenCode DESIGN.md for VibeMail Glass. Surface treatment replaced with glassmorphism. Typography replaced with JetBrains Mono (SIL OFL). All other tokens from OpenCode.'
>
> 5. Preserve the complete structure and all other sections of the original document unchanged."

### Prepare Claude Design context

```bash
mkdir -p design-context
cp docs/DESIGN_SPEC.md design-context/
cp docs/design-system/VIBEMAIL_DESIGN.md design-context/
git add docs/design-system/ design-context/
git commit -m "docs: VibeMail Glass design system — adapted from OpenCode"
```

Verify attribution, font licence, exact background values, opacity/blur ranges, and preservation of unaffected reference sections.

---

## Setting Up the Claude Design System

Create a Claude Design project named `VibeMail Glass`.

### Native design-system form

- Blurb — glassmorphic email client, deep navy, frosted glass, JetBrains Mono
- GitHub code field — leave blank during native design-system setup
- Local context — upload `design-context/`
- Font asset — upload `JetBrainsMono[wght].ttf`
- Notes — glass treatment overrides opaque reference surfaces; dense five-view email UI

After saving, link the VibeMail repository in the chat/context panel so model field names are available.

### First inbox prompt

> "Build the inbox view for VibeMail Glass as a React component structure. The design system is already configured — use it for all token values. Layout: a list of glassmorphic message rows on a deep navy background. Each row shows sender name (from), subject, snippet, timestamp, and a star icon. Unread rows are slightly more opaque with bolder subject text. Starred rows show a filled star; unstarred show an outlined star at low opacity on hover. Eight rows with realistic data. Compose button fixed to the bottom right. Search bar at the top. Label filter bar below the search bar: All, Starred, Archive, Trash chips — All selected by default. Refer to DESIGN_SPEC.md for all required states and behaviours."

Review translucency, actual JetBrains Mono rendering, information hierarchy, both star states, and the complete label navigation before iterating.

---

## Iterating Our Email UI in Claude Design

### Inbox corrections

> "The glass surfaces need to read as genuinely translucent — increase the backdrop blur and reduce the background opacity on the message row cards so the background colour shows through."

> "The message rows need stronger hierarchy. Sender name is the dominant element — full white, medium weight. Subject line slightly smaller and less bright. Snippet the dimmest of the three — around 50% opacity white."

### Thread view

> "Build the thread view as a React component structure. This replaces the inbox content area when a message row is clicked — inbox slides out, thread slides in. Thread subject as a heading at the top. Messages stacked oldest-first, matching the GET /api/v1/threads/:threadId response order. Each message is a glass card — sender avatar (initials circle) on the left, sender name and timestamp on the same line, body text below. The most recent message expanded; earlier messages collapsed to a single line with sender name, timestamp, and chevron. Back arrow at top left. Reply button at the bottom of the most recent card. Unread toggle and star icon in the top right header — star filled if isStarred. Refer to DESIGN_SPEC.md for all required states."

### Compose drawer

> "Build the compose drawer as a React component structure in three states shown side by side. State 1 — default: panel overlaying the inbox, more opaque glass background, drag handle, To field with tag-style entry, Subject, Body, Send button with accent fill, secondary Save draft button alongside Send, close button top right. State 2 — reply mode: To pre-filled and read-only, Subject reads 'Re: Morning standup notes', quoted excerpt below body at reduced opacity. State 3 — draft restored: fields pre-populated, 'Discard draft' chip below the body field at low opacity. Refer to DESIGN_SPEC.md for all error states."

### Label filter and search

> "Show the inbox with the Starred chip selected — list shows only starred messages, Starred chip has accent border and brighter fill. Also show the Archive state with no messages — 'Nothing in Archive.' centred in the list area."

> "Show the inbox in active search mode as a React component structure — search bar has a query typed, inbox list replaced by four results using the same row treatment including the star icon. Clear search button inline at right of search bar. Active search bar has a subtle accent glow. Also show the no-results state — 'No messages match your search.' centred."

### Cross-view consistency

> "Do a consistency pass across all five views. Glass card corner radius, background opacity, and border opacity should be identical for equivalent elements everywhere. The accent colour should be the same value throughout. Typography sizes for equivalent elements should match. The star icon treatment should be identical between inbox rows, search result rows, and the thread header. Flag anything inconsistent and fix it."

### Mobile layouts

> "Show me the mobile layout for all five views as React component structures — inbox, thread view, compose drawer, Starred filter active, and search active. Mobile viewport is 390px. The glassmorphic aesthetic should hold. For the compose drawer: cap the drawer at a maximum height that accounts for the keyboard and ensure the fields scroll internally so the body is always reachable."

If the body clips:

> "Cap the compose drawer at 58% of the viewport height. Fields scroll internally — body always reachable without dismissing the keyboard."

Carry three implementation flags into Claude Code: reduced motion caps blur at 8px, mobile glass borders rise to 20–25% white, and the fixed compose button respects `env(safe-area-inset-bottom)`.

---

## Auditing Our Claude Design

Audit every state in `docs/DESIGN_SPEC.md` against the canvas before export.

### Required coverage

- Inbox rows, unread/star states, skeletons, errors, empty state, pagination, filters
- Thread collapsed/expanded states, actions, skeletons, not-found error
- Compose default/reply/draft-restored modes, validation, sending, draft failures, mobile keyboard
- Label fixed/custom chips, active states, empty states
- Search activation, matching rows, clear action, keyboard shortcut, error and no-result states

### Verdict file

Create `docs/AUDIT_1B.md`:

```markdown
Approved: [what passed without changes]
One more pass: [what needs a fix before export, and what the fix is]
Cut: [anything descoped from the spec and why, or "nothing"]
```

```bash
git add docs/AUDIT_1B.md
git commit -m "docs: VibeMail Glass director's audit"
```

Only spec-compliance findings block export. Record optional polish separately.

---

## Producing a Migration Plan in Claude Code

Export the full zip, place it at `design-handoff/`, and inspect every component, the token CSS, `data.js`, and `tweaks-panel.jsx` before planning.

### Migration Plan Mode prompt

> "Migrate the Claude Design handoff in design-handoff/ into a Next.js app in app/ as a full-stack monorepo. Don't touch api/ or src/. Stack: React 19, Next.js 16, TypeScript.
>
> Key constraints: — Preserve glassmorphic CSS and adopt design token variable names from the handoff CSS tokens file — Keep data.js intact for now — Phase 1 works against sample data before wiring the real API — Convert the settings/tweaks component to persist dark/light mode, animated background (default off, respects prefers-reduced-motion), and font scaling to localStorage — The sign-in UI migrates to app/auth/page.tsx — keep the glassmorphic card and Sign in with Google button. OAuth mechanics stay in the existing api/v1/auth/google/callback.ts. Add app/auth/callback/page.tsx to read the JWT from the URL, store it, and redirect to the inbox. Redirect unauthenticated users to the sign-in page. — Inbox, thread view, and search results share a single MessageRow component — ComposeDrawer takes a mode prop: new, reply, draft-restored — Apply these three implementation flags: (1) prefers-reduced-motion caps backdrop-filter blur to 8px across all glass surfaces, (2) glass card border opacity increases to 20-25% white at mobile breakpoint, (3) fixed compose button gets padding-bottom: calc(24px + env(safe-area-inset-bottom))
>
> Use CONTRACT.md and docs/DESIGN_SPEC.md as the source of truth for all API shapes and required states. Enter Plan Mode and show me the proposed file structure before writing any files."

Do not approve until the plan has shared `MessageRow`, a mode-driven `ComposeDrawer`, persistent Settings, sample data preserved for Phase 1, and no modifications to `api/` or `src/`.

---

## Migrating the Handoff Components into a Next.js Structure

### Begin Phase 1

> "Exit Plan Mode and begin Phase 1. Migrate the handoff components into the Next.js app structure as planned. Keep data.js intact — all components render against sample data first. Build each component in sequence, converting to TypeScript as you go. Start with the shared MessageRow component and the inbox view. When the inbox renders correctly in the preview pane, stop and wait for review."

### Compose validation

> "Update ComposeDrawer.tsx to validate all three required fields before sending. Email format validation on the To field: match against a standard RFC 5321 pattern and show INVALID_RECIPIENT inline on the field if it fails. Body is required — show MISSING_FIELDS on the body field if empty. These validations run on submit, not on blur."

### Settings migration

> "Convert tweaks-panel.jsx into a Settings component at app/components/Settings.tsx. The component persists these preferences to localStorage: dark/light mode toggle (default dark), animated background toggle (default off — disabled when prefers-reduced-motion is set), and font scaling (default 100%, options 90% / 100% / 110%). Apply preferences as CSS custom properties on the document root. Wire a settings icon in the app header to open Settings as a glass overlay. At the bottom of the Settings panel, add a Sign out button that calls signOut from AuthProvider — this clears the JWT and redirects to the sign-in page."

### Mobile fixes

> "Apply these three mobile fixes: cap backdrop-filter blur to 8px when prefers-reduced-motion is set, increase glass card border opacity to 20-25% white at mobile breakpoint only, and set the fixed compose button to padding-bottom: calc(24px + env(safe-area-inset-bottom))."

```bash
git add app/ package.json
git commit -m "feat: VibeMail Glass frontend with sample data"
```

Phase 1 is complete only when all five views render and work against sample data in the preview.

---

## Visual Inspection with Our Agent and Accessibility Review

Targeted correction used in the course:

> "Increase the unread dot size — it's disappearing against the glass surface."

### Visual-consistency agent prompt

> "Look at the running app in the preview pane. Navigate through all five views in sequence: inbox (default state), thread view (click any message row), compose drawer (click the compose button), search (press Cmd+K), and the Archive label filter.
>
> At desktop viewport width, evaluate and return a structured pass/fail report:
>
> 1. Glass surfaces read as genuinely translucent — backdrop blur visible, background shows through
> 2. Glass card corner radius is consistent across all five views
> 3. Glass surface opacity is consistent for equivalent elements across all five views
> 4. Accent colour is the same hex value everywhere
> 5. JetBrains Mono is rendering — not falling back to a system monospaced font
> 6. Star icon appears in both filled and outlined states in the inbox
> 7. Unread dot is visible and uses the accent colour
> 8. Label filter bar shows All, Starred, Archive, and Trash chips
> 9. Compose button is visible and positioned at bottom right
> 10. Settings icon is present and opens the Settings panel
> 11. Dark/light mode toggle visibly changes the interface
> 12. Sign out button is present in the Settings panel
>
> Switch the preview pane to 390px mobile width and re-evaluate: 13. Message rows are at least 44px tall 14. Label filter chips scroll horizontally on overflow 15. Thread header actions do not overlap 16. Compose button is not obscured by the iOS safe area
>
> Return each item as: PASS, FAIL with description, or CANNOT VERIFY with reason."

### Accessibility agent prompt

> "Look at the running app in the preview pane. Navigate through all five views and evaluate accessibility. Return a structured pass/fail report:
>
> 1. All interactive elements are reachable by keyboard — Tab moves through inbox rows, compose button, search bar, label filter chips, and Settings icon in a logical order
> 2. The compose drawer opens with Enter or Space on the compose button and dismisses with Escape
> 3. Focus is trapped inside the compose drawer when open — Tab does not move focus behind the drawer
> 4. The Settings panel opens by keyboard and dismisses with Escape
> 5. Snippet text at 50% white opacity on the glass card background: legible or disappearing?
> 6. Star icon has a visible focus indicator when keyboard-focused
> 7. Unread dot has a visible focus indicator when keyboard-focused
> 8. Search bar activates on Cmd+K and Ctrl+K
> 9. Clear search button is reachable by keyboard when search is active
> 10. At 390px mobile: all touch targets are 44px minimum height
> 11. Reduce preview viewport height to 60% to simulate mobile keyboard — compose drawer body field remains visible and focusable
>
> For each FAIL, describe the specific element and the expected correct behaviour."

Fix failures, rerun both audits, then commit:

```bash
git add .
git commit -m "fix: visual consistency and accessibility — subagent review"
```

---

## API Integration with UI

### Contract-driven integration prompt

> "Read CONTRACT.md and DESIGN_SPEC.md. Replace data.js with real API calls across everything that has a contract endpoint. Use relative paths — the API routes are in the same repository. JWT Bearer token from localStorage. Fix any gaps where the current implementation doesn't match the contract — including type definitions, error handling, cursor pagination, server-side filtering, and auth flows. Render bodyPlain as the primary email body in thread view, with bodyHtml in a sandboxed iframe (sandbox='allow-same-origin') as a fallback when bodyPlain is null. Add a 'View in Gmail' link in the thread header alongside the unread toggle and star. The draftId field is server-managed — never send it as a client field per CLAUDE.md. Delete data.js when all references are replaced."

### OAuth and preview boundary

The embedded preview cannot complete Google OAuth. Run locally and authenticate in a real browser:

```bash
vercel dev
```

Copy the JWT from the real browser's localStorage, then set it in preview:

```javascript
localStorage.setItem('token', 'paste-your-jwt-here')
```

### API verification

- Inbox and cursor pagination
- Thread body and View in Gmail
- Read/unread and star actions from every surface
- Draft save, close, and restore
- Compose and send
- Search and server-side label filtering
- Settings sign-out and sign-in redirect

Fix every contract mismatch before continuing.

---

## Email Bodies & Attachments

### Expired-session handling

> "Add token refresh handling. When any API call returns a 401, clear the stored JWT and redirect the user to the sign-in page with a message explaining their session expired. Also wire the existing signOut function to clear the JWT and redirect on explicit sign-out."

### Plain/HTML toggle

> "Add a plain/HTML toggle button to the thread view header, alongside the unread toggle and View in Gmail link. Clicking it switches the current message between the bodyPlain render and a sandboxed bodyHtml iframe (sandbox='allow-same-origin'), defaulting to plain. The toggle state is per-message and resets when navigating to a different thread. If bodyHtml is null for a message, the HTML option is disabled."

### Update source documents before attachment work

> "Update docs/DESIGN_SPEC.md and docs/CONTRACT.md to reflect what we just built. DESIGN_SPEC.md should document the email body rendering behaviour in the thread view — plain text default, HTML toggle, View in Gmail link. CONTRACT.md should add a new section for the attachment upload endpoint we're about to build — POST /api/v1/attachments, multipart file upload, 25MB limit, returns an attachment ID. Commit both with a sensible message."

### Live inbox subscription

> "Add a Supabase Realtime subscription to the inbox. When a new message is inserted into the messages table for the current user, it should appear in the inbox immediately without a page refresh. Use the existing Supabase client — no new dependencies needed."

```bash
git add .
git commit -m "feat: Supabase Realtime subscription for live inbox updates"
```

### Attachment backend and UI

> "Implement the attachment upload endpoint at POST /api/v1/attachments per CONTRACT.md §4.11. Validate file size against 25MB — return FILE_TOO_LARGE if exceeded. Upload to Gmail using the Gmail drafts API. Return attachmentId, filename, mimeType, and size.
>
> Then add the attachment UI to ComposeDrawer.tsx per DESIGN_SPEC.md: a paperclip icon in the footer, native file picker on click, file chips below the body field with filename and remove button. Upload each file on selection via POST /api/v1/attachments. Show a spinner per chip while in flight, red error chip on failure with retry. Disable Send while any upload is in flight. Include attachmentIds in POST /api/v1/messages on send."

```bash
git add .
git commit -m "feat: attachment upload endpoint and compose drawer UI"
```

Verify a real attachment arrives in Gmail.

---

## Final Pass & Deployment

### Production visual review prompt

> "Navigate to the production app at [your Vercel URL]. Authenticate via the sign-in flow in a real browser, copy the JWT from localStorage, and set it in the preview pane console so the preview can make authenticated API calls. Then navigate through the inbox, open three different threads of varying length, open the compose drawer, activate search with a real query, and check the Archive and Trash filters.
>
> Evaluate and return a structured pass/fail report:
>
> 1. Long sender names truncate cleanly in inbox rows without breaking row height
> 2. Long subject lines truncate at one line without pushing the snippet to a second line
> 3. Thread view with many messages — collapsed rows stay at consistent height, expanded message clearly distinguished
> 4. bodyPlain renders cleanly inside the glass card — line breaks preserved, no raw HTML tags visible
> 5. The View in Gmail link is present in the thread header and visually unobtrusive
> 6. Empty state renders correctly for Archive and Trash if they have no messages
> 7. Search results with real query text render correctly — star icons present
> 8. Compose drawer reply mode shows a real quoted excerpt cleanly
> 9. At 390px mobile: a real inbox with varied content — rows consistent, no overflow
> 10. Settings panel opens and all three toggles function with real data loaded"

### Deploy fixes

```bash
git add .
git commit -m "fix: final visual pass — production review"
git push
vercel --prod
```

### Production smoke test

- OAuth in a real browser
- Inbox with real Gmail data
- Thread body, plain/HTML toggle, View in Gmail
- Reply with real quoted excerpt
- Send and attachment delivery
- Read/unread, star/unstar, archive, trash
- Draft save and restore
- Search and filters
- Dark/light mode, font scale, animated background, preference persistence
- New mail appearing without page reload
- Desktop and 390px mobile layout with no overflow

Any failure is fixed and redeployed before the final commit:

```bash
git add .
git commit -m "feat: VibeMail Glass — complete"
git push
```

---

## End-of-Module Document Set

```text
design-context/
  DESIGN_SPEC.md
  VIBEMAIL_DESIGN.md
docs/
  CONTRACT.md
  DESIGN_SPEC.md
  AUDIT_1B.md
  design-system/
    opencode.DESIGN.md
    VIBEMAIL_DESIGN.md
app/
  components/
    MessageRow.tsx
    ComposeDrawer.tsx
    Settings.tsx
```
