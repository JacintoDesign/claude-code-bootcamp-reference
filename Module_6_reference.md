# Module 6 Reference Guide
### Building a Cinematic Portfolio — Direction, Shaders, Scroll, Audit, and Ship

Use this guide alongside the Module 6 videos.

---

## Table of Contents

1. [How to Build a Cinematic Portfolio](#how-to-build-a-cinematic-portfolio)
2. [Art Direction & Motion Brief](#art-direction--motion-brief)
3. [Building the Portfolio Shell](#building-the-portfolio-shell)
4. [Understanding Shaders](#understanding-shaders)
5. [Directing a Shader Hero Section](#directing-a-shader-hero-section)
6. [Building a Control Panel for Shaders](#building-a-control-panel-for-shaders)
7. [Scroll Motion Foundations](#scroll-motion-foundations)
8. [Pinning and Scrubbing an Element](#pinning-and-scrubbing-an-element)
9. [About Section Timeline](#about-section-timeline)
10. [Working on the Project Grid](#working-on-the-project-grid)
11. [Deployment & Director's Audit](#deployment--directors-audit)
12. [Wire Up Our Contact Form](#wire-up-our-contact-form)

---

## Core Concepts

**Anatomy and treatment are separate axes.** Anatomy is the content parts: hero, about, project grid, contact, footer. Treatment is the motion attached to a part. A treatment must earn its place.

**A motion brief specifies outcomes and omits methods.** Describe what the visitor should see and feel; discover implementation values by looking.

**Two engines, distinct jobs.** Shaders create the living field. GSAP/ScrollTrigger orchestrates scroll choreography. Motion handles local component interactions.

**Uniforms are the control surface.** Time, resolution, pointer, and scroll progress are input values the agent can expose as dials.

**Visual direction is iterative.** Look, name what is off, change one thing, and look again. Notes identify a property and a direction.

**The brief revises forward, then freezes.** New earned treatments are recorded during the build. At audit time the brief stops moving so the implementation can be judged honestly.

**Reduced motion includes the shader.** Under `prefers-reduced-motion`, pinned sequences collapse, reveals become static or minimal, marquees stop, and the shader holds a still frame.

---

## How to Build a Cinematic Portfolio

### Tool stack

- Next.js 16, React 19, TypeScript
- Three.js and React Three Fiber for GLSL shaders
- GSAP, `@gsap/react`, ScrollTrigger, SplitText
- Motion from `motion/react`
- Resend for contact delivery
- GitHub and Vercel for release

### Quality rule

```text
Content part → Proposed treatment → Why it helps the visitor → Keep or cut
```

Code-native motion ships as a small recipe, reacts to pointer and scroll, and avoids generated-video weight, licensing, and fixed playback. That is useful only when the still composition, real content, and conversion path already work.

### Direction checklist

- The portfolio targets a real audience and outcome.
- Every project, description, screenshot, and link is real.
- Type is selected deliberately, with at most two families.
- The still design works before motion is added.
- Treatments are excluded as deliberately as they are included.
- Your finished visual direction diverges from the course example.

---

## Art Direction & Motion Brief

Gather references whole-to-parts:

1. Awwwards — finished-site quality and pacing
2. React Bits and Codrops — component and interaction references
3. Shadertoy — shader mood only

References are evidence, not source code. Never copy an implementation you cannot attribute, understand, or adapt safely.

### Content required before building

- Real about copy and a clear intended audience
- Three real projects with name, category/year, one-line description, stack, live URL, and consistent screenshots
- A working contact destination
- Chosen display and body type from a licence-safe source such as Google Fonts

### MOTION_BRIEF.md template

```markdown
# Motion Brief — [Portfolio name]

## Look
Feel: [two or three sentences — the sensibility every part of this page shares]
Palette: [background, foreground, one accent; exact hex values]
Type: [display face + body face, from Google Fonts]
Space: [dense or airy; wide margins or edge-to-edge; sharp or soft corners]

## Motion direction
[One or two sentences: slow and weighted, or crisp and quick; the shared rule]

## Hero
- Treatment: full-screen shader — [the target look, without implementation values]
- Treatment: [any other earned treatment]
- Why: [why each treatment helps]

  -> Transition into About: [treatment or deliberately nothing]
     Why: [reason]

## About
- Treatment: [or still, by decision]
- Why: [reason]

## Project grid
- Treatment: [treatment]
- Why: [reason]

  -> Transition into Contact: [treatment]
     Why: [reason]

## Contact
- Treatment: [or still, by decision]
- Why: [reason]

## Excluded
- [Effect considered and cut, with reason]
```

Commit `MOTION_BRIEF.md` before implementation. Exact outcomes belong here; shader octaves, speed values, and library methods do not.

---

## Building the Portfolio Shell

Create `CLAUDE.md` first with stack, naming, palette/type pointers, and this phase constraint:

```text
Current phase: STATIC BUILD
No motion of any kind. No transitions, keyframes, scroll listeners,
entrance effects, animation libraries, or animated hover treatment.
Elements render in their final state.
```

### Plan Mode prompt

> Read `MOTION_BRIEF.md` and `CONTENT.md`.
>
> Plan the static structure for this portfolio — the four content parts in page order (hero, about, project grid, contact), plus navigation.
>
> The brief assigns motion treatments to each part. Do not implement any of them. They're there to tell you where motion will live later so the structure can accommodate it. This build is static — see `CLAUDE.md`.
>
> Structure for the motion that's coming:
> - The hero is a full-viewport section containing a real, positioned, full-bleed background container. A shader mounts into that container later, so it must be a genuine element — not a background colour on the section itself.
> - The headline is a single element containing its text. It gets split into lines later, so don't build it as stacked divs.
> - Project cards are discrete, individually selectable components.
>
> Cards follow this spec: index number, name, category and year, one-line description, tech tags, link to the live deployment.
>
> All copy comes from `CONTENT.md`. Don't invent, paraphrase, or placeholder anything — if something's missing, stop and tell me. Transcribe it once into a typed content module — `lib/content.ts` — and have components read from that, so copy has one home in code.
>
> The hero background renders as a flat fill of `--bg` for now.
>
> Plan only. Don't write any files yet.

### Shell acceptance checklist

- Hero, about, project grid, contact, footer/navigation are present.
- The hero has a real full-bleed mount container.
- The headline is one splittable element.
- Cards are discrete components.
- All copy comes from `lib/content.ts` and matches `CONTENT.md`.
- Project images share an intentional aspect ratio.
- No motion exists anywhere, including CSS hover transitions.
- Desktop and mobile still composition work.

Structure is final before motion attaches; spacing and visual tuning can still evolve.

---

## Understanding Shaders

### Mental model

A fragment shader is a function evaluated for every pixel on every frame by the GPU. Noise is the raw material for organic variation. Uniforms pass changing values into the shader without rebuilding it.

| Uniform | Meaning |
|---|---|
| `uTime` | Ongoing evolution |
| `uResolution` | Canvas size and aspect correction |
| `uPointer` | Normalized visitor position |
| `uScroll` | ScrollTrigger progress |

### Performance principles

- Reuse vectors and uniforms; avoid allocating objects every frame.
- Keep shader complexity and noise octaves modest.
- Lower render resolution on smaller or slower devices.
- Mount once and update values rather than rebuilding the canvas.
- Provide a still, beautiful reduced-motion state.

The shader is directed through visible outcomes, not by guessing numeric recipes in advance.

---

## Directing a Shader Hero Section

### First implementation prompt

> Replace the hero's flat background with a full-screen shader, mounted with react-three-fiber. Uniforms for time and resolution.
>
> The look is in `MOTION_BRIEF.md`.

After the first result, make one note at a time and inspect between notes.

### Example direction notes

> Too fast. Halve the drift speed.

> The scale is too small, it's reading busy. Three or four shapes across the screen, not a dozen.

> Contrast is too high — it's reading as neon rather than deep. Pull the magenta back toward the indigo.

> It's reading as plasma. I want ink diffusing in water — slower, softer edges, colours bleeding rather than banding.

> That's close. Commit this before we go further.

### Note quality test

```text
Observable property + desired direction
```

“Make it nicer” is not actionable. “Drift is too fast; halve it” is.

Stop when the shader is clearly in the written direction and stable enough for the control-panel lesson. Do not chase final values through repeated prompt round trips.

---

## Building a Control Panel for Shaders

### Dev panel prompt

> Add a dev-only control panel for the shader uniforms — sliders for drift speed, scale and grain, colour pickers for the palette. Wire them straight into the uniforms object so changes are live. Hide it in production.

### Pointer reactivity prompt

> Add a pointer uniform to the shader — the cursor's position over the hero, normalised. Ease it toward the real cursor position rather than tracking it exactly, so the field lags slightly and settles.
>
> Add the easing amount to the dev panel.

### Control-panel checklist

- Every dial maps directly to one uniform or behaviour.
- Values update live without React rerenders on every frame.
- Touch devices have a stable fallback.
- Pointer movement is subtle and eased.
- The panel cannot appear in production.

Use the agent for structural wiring and the panel for visual decisions.

---

## Scroll Motion Foundations

Replace the static phase in `CLAUDE.md` with:

```text
Current phase: MOTION
All motion must be justified by MOTION_BRIEF.md, scoped to its component,
cleaned up on unmount, and gated for prefers-reduced-motion.
GSAP owns scroll choreography; Motion owns local component interactions.
```

### Foundation prompt

> Set up GSAP with ScrollTrigger, plus Motion (the package is `motion`, imported from `motion/react`).
>
> GSAP runs in a client component using the `useGSAP` hook from `@gsap/react`, scoped to a ref.
>
> Then add one scroll-triggered reveal: the about section heading fades and rises as it enters the viewport.

### Correct React/GSAP shape

```tsx
"use client";

const root = useRef<HTMLElement>(null);

useGSAP(() => {
  // GSAP selectors and ScrollTriggers scoped to root
}, { scope: root });
```

Prefer GSAP for timelines, pinning, scrubbing, batching, and scroll velocity. Prefer Motion for a local hover, tap, or component transition.

---

## Pinning and Scrubbing an Element

### Pinned hero prompt

> Pin the hero for one and a half viewport-heights, and feed ScrollTrigger's scroll progress into the shader as a `uScroll` uniform.
>
> As `uScroll` goes 0 to 1: travel the field vertically, deepen it, evolve the palette, and let the central glow settle back.
>
> Add two controls to the dev panel: scroll intensity — how far `uScroll` pushes the field — and the scrub lag.

### Headline reveal prompt

> Split the hero headline into lines with SplitText and reveal them with a stagger on load — the brief's “a line at a time.” The eyebrow and subhead follow, a beat behind.

Amend the brief with the wordmark treatments before implementation, then:

> Build the two wordmark treatments from the brief. Put both amounts — drift and lean — on the dev panel.

### Verification

- Pin duration helps the hero rather than trapping the visitor.
- Scroll progress is clamped and stable.
- Refresh/recalculation works after responsive layout changes.
- Headline remains readable before and after splitting.
- Reduced motion skips pinning and renders complete content.

---

## About Section Timeline

Amend the brief first: record the timeline decision and add the Ground treatment that explains what exists behind the lower page.

### Timeline prompt

> Build the about timeline from the brief. Roughly one viewport of scroll per beat, with a little scrub lag so it glides. The heading and stats stay above the pin. Copy stays in `lib/content.ts`.

### Mobile and reduced-motion fallback

> Below the desktop breakpoint and under `prefers-reduced-motion`, skip the timeline entirely — no pin, no horizontal movement; the beats render as the original vertical list.

### Count-up prompt

> The three about stats count up from zero the first time they enter the viewport. Under a second, once, never again.

### Ground and handoff prompt

> Build the ground and the handoff from the brief: as the hero unpins, the field lags the scroll slightly and recedes for the rest of the page. Render it at half resolution below the hero. Put both amounts on the panel.

Two adjacent pinned sections are the motion ceiling in the course build. Confirm the about copy stays readable and the lower-page shader no longer competes with it.

---

## Working on the Project Grid

Upgrade the card treatment in the brief before implementation.

### Batched card reveal

> Build the card reveal from the brief. Use `ScrollTrigger.batch` so the cards arrive as a wave.

### Card hover

> Add a hover to the project cards with Motion: a small calm lift, the cyan accent picking out the border, a very slight zoom on the screenshot. Nothing else moves.

Add the Footer treatment to the brief, then:

> Build the footer marquee from the brief.

Add the Site-wide treatment to the brief, then:

> Build the Site-wide treatments from the brief, and nothing else. The Excluded list still stands.

### Grid checks

- Screenshots are the visual evidence and remain dominant.
- Every card is real, accurate, and linked.
- Hover exists only for interactive cards and has keyboard-equivalent focus treatment.
- Batch reveal preserves readable document order.
- Marquee starts after the main conversion point and can stop under reduced motion.
- Site-wide polish does not smuggle excluded effects back into the build.

---

## Deployment & Director's Audit

### Reconciliation prompt

> List every animation, transition and hover in this codebase — component by component, one line each, including CSS-only ones. Then check each against `MOTION_BRIEF.md` and tell me which ones the brief doesn't mention.

For each undocumented treatment, decide:

- Add it to the brief because it earns its place.
- Add a missing reduced-motion gate in code.
- Qualify an over-broad brief rule.
- Remove it because it conflicts with the direction.
- Record an intentional disagreement as an audit finding.

When reconciliation is complete, freeze `MOTION_BRIEF.md`.

### Mechanical audit prompt

> Audit this codebase against `MOTION_BRIEF.md` and report as a list:
>
> - Any animation or transition with no reduced-motion gate.
> - Anything that can reach production but shouldn't — dev panels, debug flags, stray console output.
> - Any copy not coming from `lib/content.ts`, or that reads as placeholder.
> - Any internal or external link that 404s.
>
> Findings only. Don't fix anything.

### Human audit

Walk hero, about, project grid, contact, footer, and site-wide treatments. Ask whether each treatment earns its place, whether pinning serves rather than traps, whether pointer response is subtle, whether still sections feel intentional, and whether the final hero matches the written look.

### Performance and accessibility

- Test GPU and main-thread performance on a real mid-range phone.
- Check long-lived work: shader loop, ticker, marquee, scroll velocity, and triggers.
- Enable reduced motion and confirm all content remains usable.
- Confirm the shader holds a still frame.
- Pause or simplify persistent work when offscreen or below the hero.

### AUDIT_4.md

Record each part's treatment, pass/cut decision, and reasoning. Include:

- Hero fidelity to the brief
- Performance on a real device
- Reduced-motion result
- Changes made from findings
- How the brief moved: additions, overturned decisions, and undocumented treatments found during reconciliation

The deployment itself is not verified until the live site is walked on desktop and phone, WebGL renders, production motion fires, links resolve, and the contact path delivers.

---

## Wire Up Our Contact Form

### Local environment

```dotenv
# .env.local — ignored
RESEND_API_KEY=
CONTACT_TO_EMAIL=
```

Add the same values in Vercel and restart the local development server after changing `.env.local`.

### Contact form prompt

> Wire the contact form to Resend with a server action.
>
> - Validate name, email and message server-side; return field-level errors.
> - Send from my Resend sending address with the visitor's in `replyTo` — never as `from`.
> - Add a honeypot field; silently drop anything that fills it.
> - Report sending, sent and failed states inline.
> - `RESEND_API_KEY` from the environment.

Send as an authenticated address and put the visitor in `replyTo`. If you do not own a domain, Resend's default sender can deliver to the address on the same Resend account. Test the actual inbox and the reply destination, not only the provider dashboard.

### Generated favicon

Delete the scaffolded `app/favicon.ico` first because it overrides generated metadata icons.

> Generate the favicon in code — `app/icon.tsx` with `ImageResponse` from `next/og`. A single bold J in `--bg`, centred on a `--violet` circle. Use the default font; don't import one. Add an apple-icon too.

Inspect the icon at real tab size.

### Ship checks

```bash
git status
npm run build
git push
vercel --prod
```

On the exact production URL, verify:

- WebGL shader renders rather than failing silently.
- Scroll sequences and pointer treatments work after production optimization.
- Reduced-motion mode is complete and usable.
- Contact submission reaches your inbox and Reply goes to the visitor.
- Environment variables are present in Production.
- Generated favicon appears in a fresh browser tab.
- Every project and navigation link resolves.
- Desktop and phone maintain smooth performance and readable composition.

---

## The Prompt Pattern Across the Module

The first shell prompt is long because no shared implementation vocabulary exists yet. After the brief is complete, prompts carry only the method delta:

```text
Document holds the outcome
Prompt names the implementation delta
Visual review supplies the next directional note
Audit reconciles code back to the frozen document
```

The quality of the prompts is downstream of the quality of `MOTION_BRIEF.md`, `CLAUDE.md`, and `CONTENT.md`.
