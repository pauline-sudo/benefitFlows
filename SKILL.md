---
name: payflip-benefit-flow
description: >
  Payflip's "Consistent benefit template" — the 4-step flow pattern (Intro / Configure / Review /
  Success) every benefit in the employee app must follow. Covers pension savings, bike leasing,
  mobility reimbursement, housing cost, multimedia, Alan insurance, warrants, extra holidays,
  private health insurance, family member health insurance, ceiling insurance, and L&D
  reimbursement. Use whenever building, prototyping, or critiquing a Payflip benefit flow on app
  or web — including HTML/React prototypes, Figma-to-code, design and copy reviews, mobile
  redesigns, or new benefit rollouts. Trigger when the user mentions any benefit by name, says
  "use our benefit pattern," asks to prototype a Payflip employee-facing flow, or starts a
  conversation about flows in the Payflip employee app. Compose with payflip-brand-voice for
  user-facing copy.
---

# Payflip benefit flow design system

## Source of truth

The canonical specification lives in the Notion page **"🧩 Consistent benefit template"** in the Employee app teamspace. This skill is the tactical companion to that document — it captures the same rules in a form that's actionable inside a Claude chat, plus the testing-driven failure modes and the briefing template that make new flows fast to build.

**When in doubt, the Notion page wins.** This skill should be updated whenever the Notion page is.

Each individual benefit (Pension savings, Bike leasing, etc.) has its own sub-page in Notion with the specific configure-step layout, copy, and edge cases. Always read the relevant benefit's sub-page before starting a flow. The Figma file referenced in Notion has 48+ screens across the 13 benefit types — pull design context from there directly when building.

---

## Part 1 — The 4-step flow (every benefit follows this shape)

Every benefit uses the same 4 steps in the same order. Layout, component order, and information hierarchy are identical across benefits. Only the **content within Configure** varies (see Part 3).

### Step 1: Intro

The landing screen. Job: help the employee quickly understand what this benefit is and whether it's relevant.

**Components in order:**
1. `FlowHeader` — back arrow + benefit title
2. `BenefitHeader` — shared component, accepts emoji icon, benefit type badge, title, description as props. No per-benefit variants.
3. `BudgetPills` — which budget(s) fund this benefit
4. `ValueCard` — *optional*; 3–5 key facts as icon + text rows. Skip when the financial preview alone carries the value.
5. **Financial preview card** — headline showing max value (e.g. *"If you convert the max → ~€7.728 net"*). Optional one-line explainer beneath (e.g. *"Net amount on your next payslip"*).
6. **More info link** — points to the benefit's Intercom article when one exists. Used as the "More info" entry point on the intro page. Skip if no Intercom article exists for this benefit.
7. **T&C link** — discoverable on intro for awareness (link only, not the checkbox). The checkbox itself lives on review (Step 3).
8. `StickyFooter` — primary CTA "Choose this benefit"

**Design rules:**
- Description max 2 lines, written for someone who has never seen this benefit
- **Intro must be scannable without interaction** — financial preview, value, budget pills, headline highlights, and CTA are always visible flat
- **Use accordion (not collapsible)** for supplementary context — FAQs, regulatory documents (IPID/KIID/DIP for insurance), "How to claim" details. Accordions never hold decision-critical content.
- **No HowItWorks** — process steps live on the review screen as "What happens after you confirm"
- Financial preview format is content-driven — use whatever structure makes the value legible for that benefit
- For benefits where the value isn't financial (extra holidays, multimedia products) — skip the financial preview entirely
- **T&C: link on intro, checkbox on review.** The link on intro is for awareness ("you can read these now if you want"); the checkbox on review is the gate.

### Step 2: Configure

The step where the employee makes their actual choice. This is the most variable step — see Part 3 for the full configure-variants table.

**Components in order:**
1. `FlowHeader` — back arrow + step title
2. `Stepper` — *if* configure has 2+ sub-steps; otherwise omit
3. `LivePreview` — persistent affordance showing the consequence of the choices. App: sticky row above StickyFooter (compact: label + amount + optional advantage line). Web: side panel (full breakdown). Updates live, persists across all configure sub-steps.
4. **Benefit-specific input** — see configure variants table
5. **Budget source label** — subtle text showing which budgets fund this and the max
6. `StickyFooter` — Continue button with running amount

**Design rules:**
- LivePreview shows label + primary value + optional advantage line ("↗ Advantage €X vs cash"). Skip the advantage line for benefits where it's not meaningful (extra holidays, bike lease).
- Show MAX button where applicable (warrants, holidays, pension)
- Validation errors appear inline, not as toasts after submission
- For external providers (Coolblue, O2O, Alan): show budget context **before** the redirect
- **No standalone collapsibles** — tax breakdown is inline inside the hero card
- **No HowItWorks**
- **`BudgetSelector` is *always* its own configure sub-step**, never combined with other inputs (resolved May 2026 design sync). Single eligible budget → render as pre-selected rich radio so the employee still sees the budget label and available amount.
- **Budget visibility on input sub-steps** — because budget selection is now separated, the input sub-step must surface budget context proactively. Use four layers:
  1. **Available-budget header** at the top of every input sub-step: small uppercase line "YOU HAVE €X AVAILABLE" + "across your [budget names]". Sum of all eligible budgets. No interaction — just sets the ceiling.
  2. **LivePreview shows running total against budget** in its subtitle: "€540 of €2,150 available". Goes amber at >80% of total. Goes warning-text when exceeded ("€2,400 — exceeds your €2,150 available") but doesn't block.
  3. **MAX button respects budget**: caps at `min(legalMax, totalAvailableBudget)`. When MAX caps below legal max because of budget, surface the reason in a tooltip or inline note ("Capped at X — your available budget covers up to that").
  4. **Soft inline warning when input exceeds total budget**, above the Continue button: "⚠ You don't have enough budget for this. Continue to the budget step to split across pots, or reduce your selection." Continue stays enabled. The budget step handles the multi-source split mechanics.
- **Only show eligible budgets** for the specific benefit. Don't render disabled "you can't use this for this benefit" options — filter them out entirely. Eligibility comes from the benefit config (Section 8 of the briefing).
- **Multi-source split rule**: if the user's chosen primary budget covers the full cost, the secondary options stay disabled. Splitting only activates when the primary is insufficient (on-click reveals a checkbox and the secondary radio group). Don't pre-show the split UI when it isn't needed.
- **Single-option pattern**: if a select/dropdown has only one option, render as a static card (preselected, non-clickable) instead of a select. Applies to budget selectors, category selectors, provider lists, anything that's normally a multi-option list.
- **"vs cash" comparisons** are appropriate on intro and configure (motivation). On review: avoid implying unspent budget defaults to cash.
- **Reimbursement flows always start with file upload as a dedicated sub-step.** OCR reads the document and prefills the amount on the next screen. If OCR fails, inline `Message` component on Verify amount screen — user can still proceed manually; submission flagged for admin review.
- **Stepper rule**: use when configure has 2+ distinct sequential screens. Most reimbursement flows are 3 sub-steps (Upload → Verify → Budget). Insurance enrollment can run 4–6. If 5+, document why each one can't be combined.
- **`TermsCheckbox` always lives on review, never on configure.**

### Step 3: Review

Confirmation screen before final submission. Everything the employee is about to commit to.

**Components in order:**
1. `FlowHeader` — back arrow + "Review your choice"
2. **Editable summary cards** — one per major decision (e.g. Amount / Document / Budget for pension savings; Days / Budget for extra holidays). Each card shows label + value + small Edit button. Edit opens a **focused modal** for that one field, never navigates back to configure (see "Edit modals" rule below).
3. **Financial summary card** — ONE lightweight card. Benefit name + total amount + budget label + key advantage in one line. **No redundant detail** — full breakdown lives on configure.
4. `Timeline` card — "What happens after you confirm". *Required* for choice flows (warrants, holidays) and external provider flows (bike, multimedia, Alan). *Optional* for simple reimbursement flows (pension, mobility, health, L&D, housing) where success-screen timeline is sufficient.
5. `TermsCheckbox` — agree to T&C with link to full terms. Stays above the sticky footer, not inside it.
6. **Sticky footer (review pattern)** — differs from configure. App: deduction-impact line ("€X will be deducted from your [budget] upon confirmation") above full-width CTA. Web: CTA in side panel, no sticky footer.

**Design rules:**
- **Edit modals (NOT navigate-back).** Edit buttons on review cards open focused modals for the specific field — `EditAmountModal`, `EditDocumentModal`, `EditBudgetModal`, etc. Never send the user back to a configure sub-step to make a small change. Reasons: keeps the user in commit context, scopes the edit to one decision, avoids re-traversing the flow. Pattern:
  - Each card on review has its own modal
  - Modal contains only the input(s) for that field, plus Save and Cancel
  - Modal handles its own validation (ceilings, caps, required fields)
  - On Save: modal closes, review re-renders with recalculated totals (totalImpact, budget allocation, etc.)
  - If a cascade creates a new issue (e.g., raised amount now exceeds primary budget), it surfaces on the affected review card — user clicks Edit on that card to resolve. No back-navigation needed.
- Timeline must include at minimum: submission → processing/review → activation → expiry/renewal
- Timeline uses **gray dots** (nothing completed yet — user hasn't clicked confirm)
- First step says "After you click confirm below" — not "Just now"
- **Always show benefit value vs. cash** on review and in active choices — core Payflip USP
- **No cash remainder or unspent-budget comparisons** ("vs retail price" is fine)
- **One-time vs spread benefits** affect what review shows:
  - **One-time** (pension savings, warrants, extra holidays): single deduction shown directly. No timing dropdown.
  - **Spread** (multimedia, mobility recurring, ceiling insurance, family insurance): dropdown on review for the timing/spreading detail (e.g. "12 monthly payments of €X starting next month"). User confirms the spread schedule before submitting.

### Step 4: Success

Confirmation. Tells the employee exactly what happens next.

**Components in order:**
1. **Status bar** — time only (no back arrow — terminal screen)
2. **Success icon** — 64×64px green circle with checkmark, centered
3. **Title** — flow-type-specific (see Part 2)
4. **Subtitle** — 2–3 lines: amount + benefit name + status
5. `Timeline` — same timeline as review, first step now green + "Just now"
6. **Buttons** — primary ("View your [benefit]") + secondary ("Back to benefits")

---

## Part 2 — Flow types (categories for grouping benefits, NOT for varying copy)

Benefits group into four categories based on what kind of decision the user is making. **All flow types share the same success title pattern (`[Benefit name] submitted!`) and the same review CTA (`Submit choice`).** Flow type informs your design *thinking* (how to frame the value, what edge cases to plan for) — not the literal verbs you put on screen.

| Flow type | Benefits |
|---|---|
| **Reimbursement** | Pension savings, mobility reimbursement, private health, L&D reimbursement, housing cost |
| **Direct choice** | Warrants, extra holidays |
| **External provider** | Bike leasing (O2O), multimedia (Coolblue), Alan insurance |
| **Insurance enrollment** | Ceiling insurance (Liantis), family member health insurance |

**Universal copy across all flow types:**
- **Success title**: `[Benefit name] submitted!` (e.g. "Pension savings submitted!", "Extra holidays submitted!", "Bike lease submitted!", "Liantis ceiling insurance submitted!")
- **Review CTA**: `Submit choice` — the deduction-impact line above the button carries the amount, so the button label stays clean
- **Cancel action**: `Cancel choice`
- **View action on benefit detail entry point**: `View choice`
- **Status copy**: "your choice is being reviewed" / "your choice is active"

**Subtitle on success** (varies by flow content, not by flow type):
- *"€990 pension savings · under review by admin"*
- *"3 days · awaiting HR confirmation"*
- *"Cowboy Classic · awaiting admin approval"*
- *"Liantis flex health · €1,500 ceiling · coverage starts Jan 1"*

**Why universal verbs:** testing showed inconsistent verbs by flow type added cognitive load without clarifying anything. The benefit name in the title carries the per-type context naturally. Resolved May 2026.

---

## Part 3 — Configure variants (the deepest variation point)

This is where benefits diverge. Identify the variant from this table; the wrapper components stay the same.

| Benefit | Input pattern | Configure sub-steps |
|---|---|---|
| **Warrants** | Number stepper + price calculator | 1: Stepper → 2: Budget |
| **Extra holidays** | Number stepper + day-value calculator | 1: Stepper → 2: Budget |
| **Pension savings** | Document upload + OCR | Upload → Verify amount → Budget |
| **Reimbursement (private health, L&D, housing)** | File upload + OCR | Upload → Verify amount → Budget |
| **Mobility reimbursement** | Document upload + OCR + category selector + amount | Upload → Verify + category → Budget *(mobility budget only; recurring submissions throughout the year)* |
| **Housing cost (mortgage/rent)** | File upload + OCR | Upload → Verify amount + type toggle + date range + non-duplication checkbox → Budget |
| **L&D reimbursement** | File upload + OCR | Upload → Verify amount + category (both OCR-prefilled) → Budget |
| **Bike lease** | External redirect to O2O | Show budget context → Redirect |
| **Multimedia** | Popular picks grid + Coolblue redirect | Browse → Show budget context → Redirect |
| **Alan insurance** | External redirect to Alan | Show budget context → Redirect → Plan summary returned |
| **Ceiling insurance (Liantis)** | Slider with live cost preview | Ceiling slider → Personal details → Address → Family members → Budget |
| **Family member health insurance** | Adult/children stepper | Stepper → Personal details → Budget |

**Reading this table when designing a new benefit:**

1. Find the closest match in the table.
2. The "input pattern" column tells you which UI primitive drives the screen.
3. The "configure sub-steps" column tells you the stepper sequence.
4. If the benefit doesn't fit cleanly, read the closest match's Notion sub-page first — there may be edge cases already documented (e.g. ceiling insurance's multi-step enrollment is the canonical pattern for any insurance flow needing identity/address/dependant data).

**For reimbursement flows specifically** (the upload-first pattern):
- **Success state**: amount silently prefilled. Description: *"We read this amount from your document. Change it if you'd like to claim less."*
- **Failure state**: inline `Message` component, user can proceed manually. Submission flagged on admin side for manual review.

**Universal rule (resolved May 2026):** every benefit's configure has at least 2 sub-steps when an input precedes the budget — input(s) first, budget second. Extra holidays and Warrants previously rendered both on one screen as "co-inputs"; that's now superseded. The Notion sub-pages for those benefits need updating to match.

---

## Part 4 — Component vocabulary (canonical names from the codebase)

Use these names when prompting Claude or briefing engineers. Never invent new names — Notion + the codebase already settled this.

**Where these live:** `@payflip/ui`. The package is built on top of shadcn/ui primitives (Radix + Tailwind), so anything in `@payflip/ui` is already styled to Payflip's design tokens. When a component the flow needs *isn't* in `@payflip/ui` yet, fall back to the underlying shadcn primitive (see Part 11 for the priority order).

| Component | Purpose |
|---|---|
| `Card` | Generic surface |
| `Btn` | Buttons (primary, secondary, ghost, link) |
| `BenefitHeader` | Intro header (icon + badge + title + description) |
| `FlowHeader` | Per-screen header (back arrow + step title) |
| `BudgetPills` | Which budgets this benefit uses, on intro |
| `BudgetSelector` | Configure sub-step for choosing funding source(s) |
| `HighlightsCard` | Key facts on intro |
| `ValueCard` | 3–5 key facts as icon + text rows |
| `LivePreview` | Persistent live-updating preview of the user's choices |
| `StickyFooter` | Bottom-pinned action bar (app) |
| `Stepper` | Multi-sub-step progress indicator |
| `TermsCheckbox` | T&C agreement, on review |
| `Timeline` | Status timeline used on review and success |
| `SuccessScreen` | Step 4 wrapper |
| `HowItWorks` | *Currently not used anywhere* — kept in the library for legacy reasons |

---

## Part 5 — Visual system

**Foundation:** Payflip's design system is built on shadcn/ui (Radix + Tailwind). The components in `@payflip/ui` wrap shadcn primitives with Payflip's tokens applied. Tokens below are the source of truth; shadcn's semantic variables (`--primary`, `--background`, etc.) alias these so primitives inherit Payflip styling automatically.

### Color tokens (use these names, never hex codes)

- `--color-primary` (vivid pink) — **interactive only**: button backgrounds, focus rings, active drop zones, selected radio dots, checkbox checked states. **Never use for decoration.** Testing has shown users misread pink decorative borders as buttons.
- `--color-brand-text` (deep brand pink) — **decorative only**: icons next to brand labels, ✦ section heading prefixes, advantage labels
- `--color-brand-bg` / `--color-brand-bg-deep` — soft pink backgrounds and non-clickable card borders
- `--color-text-prominent` / `--color-text-default` / `--color-text-muted` — text hierarchy
- `--color-surface` / `--color-border-light` / `--color-border-mid` — neutral surfaces and dividers
- Success / warning / info palettes — for status states only, never for branding

### Typography

- Display/headings: **Studio Feixen Sans** (`ff-display`), bold for headlines, semibold for section headings
- Body: **Poppins** (`ff-poppins`), regular for body, medium for emphasis, semibold for action labels
- Numbers: `tabular-nums` for any €amount, rates, or counts

### Spacing

- App: 20px outer gutters
- Web at `lg`+: 24px outer gutters, two-column ~680px content + ~320px sticky side panel, 1200px max-width container at `xl`+
- Cards keep their internal padding regardless of breakpoint

---

## Part 6 — Web vs app patterns

Both surfaces use the same 4-step logic and the same shared components. Differences are structural (screen size, navigation), not content.

| Pattern | App | Web |
|---|---|---|
| Step navigation | Sequential screens, one step per screen, back/forward | Single page, all steps rendered inline, scrolled through |
| Stepper | Use when configure has 2+ sequential screens | No stepper — section headers separate steps |
| BudgetSelector placement | Configure sub-step (final) | Configure sub-step (final) |
| Layout | Single column, `StickyFooter` pinned to bottom | Two-column, scrolling content left, sticky summary side panel right |
| LivePreview | Sticky row above CTA, persists across configure sub-steps | Side panel content, full breakdown, persists across configure and review |
| Sticky footer (configure) | LivePreview row + half-width Continue | N/A |
| Sticky footer (review) | Deduction-impact line + full-width primary CTA | CTA in side panel, no sticky footer |

### Web breakpoints

- `sm` (768px): follows app pattern — sequential, FlowHeader + Stepper + StickyFooter, no side panel. **Below 768px is not supported** — redirect to native app.
- `md` (1024px): single-column, side panel collapses below content or into a drawer
- `lg` (1280px): **two-column activates** — primary desktop target
- `xl` (1440px): max-width container kicks in (1200px), margins increase

### What stays identical on both surfaces

Component names, shared component props, copy rules, button count on success, BudgetPills on intro, BenefitHeader, TermsCheckbox on review, Timeline on success, success titles by flow type.

---

## Part 7 — Per-benefit briefing template

Fill this out before starting any new benefit-flow chat. Most "Claude got the design wrong" moments trace back to a briefing gap.

```markdown
# Benefit briefing: [BENEFIT NAME]

## 0. Prototype path (decide first — affects everything below)
- [ ] **HTML throwaway** — single self-contained file with React via CDN.
      Use when: the pattern is novel and you need fast iteration, or the
      flow needs heavy bespoke prototyping for user testing. Expect visual
      drift from production. Plan to rebuild later.
- [ ] **Storybook composition** — thin .tsx file importing real components
      from `@payflip/ui`. Use when: the archetype is settled (existing
      reimbursement, existing direct-choice, etc.) and you want production
      parity. ~200 lines of composition, no component re-implementation.

If HTML: reference pension savings prototype as the visual baseline.
If Storybook: see Part 11 for composition patterns and import map.

## 1. Flow type
Which flow type from Part 2?
- [ ] Reimbursement
- [ ] Direct choice
- [ ] External provider
- [ ] Insurance enrollment
- [ ] Other (describe — likely needs Notion sub-page documentation)

## 2. One-sentence purpose
What does this benefit do for an employee, in plain words?
(e.g., "Reimburses last year's pension savings contributions via the next payslip.")

## 3. Configure variant
Which row from Part 3's variants table is closest? Or describe a new pattern.

## 4. Configure sub-steps (if multi-step)
List the sub-steps in order. State why each can't be combined with the next.
(e.g., "Upload — needs OCR pass before next screen. Verify amount — user confirms or corrects. Budget — funding selection.")

## 5. Temporal framing
- [ ] Past (reimbursement of something already done — use past tense in copy)
- [ ] Present (configuring an ongoing benefit — present tense)
- [ ] Future (will start at a defined point — future tense, give the start date)

## 6. Inputs the user provides
- [ ] Document upload (specify type, where users find it — populates the help modal)
- [ ] Numerical input (specify: amount? quantity? frequency?)
- [ ] List selection (specify the list — providers, vendors, categories)
- [ ] Personal info (specify — subscription number, address, dependants)
- [ ] None — just confirmation

## 7. Math
What number does the user enter / is read?
What's the legal ceiling, if any?
What's the employer cost rate? (Default 8.86% for Belgian benefits — verify per benefit.)
What's the net advantage rate vs taking the equivalent as cash?

If math doesn't reconcile cleanly, label deductions broadly ("Tax & social charges") rather than specifically.

## 8. Funding sources
Which budget pots can fund this benefit?
- [ ] Bonus
- [ ] End-of-year premium
- [ ] 13th month
- [ ] Mobility budget (some benefits are mobility-only — e.g. mobility reimbursement)
- [ ] Other: ___

Are there benefit-specific restrictions?

## 9. Recurrence
- [ ] One-shot annual (pension savings)
- [ ] Recurring monthly (multimedia, ceiling insurance premium)
- [ ] Configurable then ongoing (meal vouchers, family insurance)
- [ ] Multiple submissions per year (mobility reimbursement)

If recurring, specify the cycle, cancellation window, and whether parameters can change mid-cycle.

## 10. External integrations
Does this benefit redirect to a third party (O2O, Coolblue, Alan, Liantis)?
If yes: where in the flow does the redirect happen, what does the user bring back, and what's the failure mode if the third party is unreachable?

## 11. Status states for benefit detail
After submission, which states does the benefit go through?
What action can the user take in each state (cancel? edit? renew?)?

## 12. Regulatory disclosure documents
Does this benefit need IPID / KIID / DIP fiches? (Insurance benefits typically do.)
If yes: where do they live (intro accordion vs review near TermsCheckbox)? Confirm with legal.

## 13. Test priorities
Top 2–3 things you most want to validate with users on this flow.
Used to write the test script.
```

### Standard kickoff prompt

```
I'm building a [BENEFIT NAME] flow following our Payflip benefit pattern.
Apply the payflip-benefit-flow skill.

Here's the briefing:

[paste filled-out template]

Build the prototype as a single self-contained HTML file with React via CDN
(same setup as our pension savings prototype). Match the visual system,
mobile patterns, and copy rules in the skill. Use flow type [X] and configure
variant [Y] from the skill.

If the briefing has gaps that block you, ask. Don't ask about anything that's
covered by the skill or the Notion source.
```

---

## Part 8 — Failure modes seen in testing (carry these forward)

These came out of pension savings testing and design review. Apply defensively to every new flow.

1. **"Is this starting a new thing or reimbursing an existing one?"** — for reimbursement flows, step 1 of "How it works" *was* on the intro and got removed. Now the explicit past tense is in the BenefitHeader description. If users still misread the temporal frame, escalate.
2. **Budget impact > user's input amount** (because of employer cost) — users default to thinking the lower number is what's deducted. Always show the inline breakdown on the budget configure sub-step's LivePreview. Never just show the single total without the contribution + employer-cost breakdown when they differ.
3. **Submit button location** (review side panel on web, sticky footer on app) — don't put a redundant Submit at the form bottom. Trust the LivePreview/StickyFooter pattern.
4. **Pink primary color on non-clickable elements** — never use `--color-primary` for decoration. Decorative pink is `--color-brand-text` or `--color-brand-bg-deep`.
5. **Disabled buttons on T&C unchecked** — don't disable. Show inline error on click. Disabled buttons hide the failure mode.
6. **Modals on mobile feeling like desktop popovers** — use bottom drawer pattern (rounded top corners, drag handle, slide-up animation, max-height 85%).
7. **Stepper not animating on step change** — render Stepper outside any keyed wrapper that remounts (in app shell, not in the page-transition div).
8. **Decorative card borders in primary pink** — use `--color-brand-bg-deep`. Tested: users hover/tap pink-bordered cards expecting interaction.
9. **Review step appearing in stepper** — testers expect more form work after Review. Stepper covers configure only; Review is the un-stepped commit moment.
10. **"vs cash" comparisons on Review** — implies unspent budget defaults to cash. Stick to "vs retail price" if comparison is needed.
11. **Visual drift between benefit flows** — when each benefit is a self-contained HTML file, Claude re-implements components from memory, not from Storybook. Drift is inevitable. Fix: use Storybook composition mode (Part 11) for any benefit that's a variant of a settled archetype. Reserve HTML mode for genuinely novel patterns.

---

## Part 9 — Open questions (mirrors Notion's, plus testing-driven additions)

These are unresolved decisions. When you hit one in a real flow, document the resolution back into Notion AND this skill.

### From the Notion page
1. **Should "How it works" appear anywhere?** Currently nowhere. Some benefits (warrants/tOption, bike via O2O, Alan) have unusual processes. Subtle note? Collapsible? *Open.*
2. **Is the intro too sparse?** No financial preview on holidays. No "What you'll need" line for benefits requiring documents. *Open.*
3. **Financial preview on intro — heuristic for when to show?** Show when financial benefit is the primary value prop, skip when not. Edge cases: multimedia, bike. *Open.*
4. **Timeline placement — review vs success?** Currently both. Risk: clutters review. *Open.*
5. **Regulatory disclosure document placement** (IPID, KIID, DIP) — intro accordion vs review near TermsCheckbox vs both? *Pending legal review.*

### Resolved May 2026
6. ~~**"Claim" vs "Choice" wording.**~~ **Resolved May 2026.** Universal "Submit choice" CTA + "[Benefit name] submitted!" success title for all flow types. Notion + this skill aligned. Verb choice doesn't vary by flow type — only by interaction context (Submit vs Cancel vs View). The benefit name carries per-type meaning naturally.

### From pension savings testing (add as more benefits are tested)
7. **Submit button discoverability on web review** — testing showed time-to-find varies by tester. Threshold: if ≥10s for any tester, add a secondary inline CTA at form bottom.
8. **Mobile chevron/details toggle discoverability** — pension savings predicted 0/1 voluntary tap. If confirmed: inline the breakdown instead of toggling.

### Resolved May 2026 design sync
9. ~~**"Budgets & advantage" as separate step**~~ **Resolved May 2026: Reading A.** Budget selection is *always* its own configure sub-step — never combined with other inputs, even for simple benefits like Extra holidays. To address the resulting awareness gap (user picking quantity without seeing budget context), see the **"Budget visibility on input sub-steps"** rule under Configure design rules.

   Implication: the Extra holidays Notion sub-page needs updating — its current "co-inputs on one screen" decision is overridden. Configure variants table should show 2 sub-steps for Extra holidays (Stepper → Budget) and similarly for Warrants.

---

## Part 10 — How to use this skill in practice

### Setup once

1. **Create a Claude Project** ("Payflip benefit flows" or similar)
2. **Drop into project files**:
   - This `SKILL.md`
   - The current pension savings prototype HTML (as a reference of the patterns in working code)
   - The pension savings test script (as a template)
   - Optionally: link to the Notion page in project instructions
3. **In project instructions**, write something like:
   > Always apply the payflip-benefit-flow skill. When briefing a new benefit, expect a filled-out template (Part 7). Read the relevant Notion benefit sub-page before starting.

### Per benefit flow

1. Open a new chat in the project
2. Fill out the briefing template (Part 7)
3. Use the standard kickoff prompt
4. Pull Figma context via the Figma MCP if there are existing designs (the file with 48+ benefit screens)
5. Iterate
6. When test sessions surface new failure modes, document them in Part 8 of this skill

### Composing with other skills

- **Always pair with `payflip-brand-voice`** for any user-facing copy — the brand voice handles tone; this skill handles structure
- **Use `frontend-design`** for any visual polish or component-level refinement work
- For testing setup, reference the pension savings test script's hypothesis-prioritization pattern

### When the pattern doesn't fit

Some benefits will need a structurally different shape (real-time approval flows, multi-party flows involving managers, etc.). When this happens:
1. Document the mismatch in the briefing
2. Propose a new variant
3. Resolve with Filip / the design team
4. Update the Notion page first, then this skill

---

## Part 11 — Composing from Storybook (the canonical path for variants of known archetypes)

When the briefing's section 0 says "Storybook composition," Claude does NOT re-implement components. It imports them from `@payflip/ui` and composes a thin benefit flow.

### Why this matters

Re-implementing components in self-contained HTML drifts visually, every time. Pension savings + Extra holidays are 95% identical visually but the 5% drift adds up — different padding, slightly off icon sizes, custom radio variants instead of the canonical RichRadio. Each new HTML benefit flow makes drift worse.

The fix: components live in Storybook (the source of truth), prototypes import from there. Visual updates to a component cascade to every benefit flow automatically. No re-implementation, no drift.

### Priority order when a flow needs a component

1. **`@payflip/ui` first** — canonical, branded. Always check here.
2. **shadcn primitive as fallback** — `@payflip/ui` is built on shadcn, so primitives share the foundation. Use shadcn directly when the equivalent isn't in `@payflip/ui` yet.
3. **Custom build only as last resort** — and only when neither has what's needed.

When step 2 happens, that's a signal: the shadcn primitive being used is a candidate for promotion into `@payflip/ui`. Add a comment like `// FALLBACK: candidate for @payflip/ui` next to the import so engineering can scan for what's missing.

**Never reinvent shadcn primitives.** If a flow needs a Tooltip, Accordion, Popover, Dialog, Sheet, Alert, Toggle, etc., and `@payflip/ui` doesn't expose it yet, import from shadcn — don't write a custom version. Custom only happens when neither layer has the pattern (e.g., NumberStepper, Stepper, Timeline are genuinely Payflip-specific compositions).

### Import map

The canonical Payflip components live in `@payflip/ui`. When a needed component isn't there yet, fall back to the shadcn primitive in `@/components/ui/*`.

```tsx
// Layer 2 — Payflip components (canonical, branded, always prefer)
import {
  // Layout shell
  TopNav, MobileFlowHeader, FlowFrame, FlowActions,
  // Step structure
  Stepper, StickyFooter, BenefitHeader, BudgetPills,
  // Configure inputs
  NumberStepper, BudgetSelector, RichRadio, FileUpload,
  // Outcomes & previews
  LivePreview, ValueCard, HighlightsCard, AdvantageHero, AdvantageChip,
  // Review + Success
  CollapsedCard, Timeline, TermsCheckbox, SuccessScreen,
} from '@payflip/ui';

// Layer 1 — shadcn fallbacks (only when @payflip/ui doesn't have it yet)
// FALLBACK: candidate for @payflip/ui
import { Tooltip } from '@/components/ui/tooltip';
// FALLBACK: candidate for @payflip/ui
import { Accordion } from '@/components/ui/accordion';

// Utilities
import { cn } from '@/lib/utils';
```

Adjust subpaths to match your actual repo structure. The point is the **two-tier hierarchy** — `@payflip/ui` first, shadcn fallback marked with a flag comment.

### Thin benefit flow file shape (~200 lines, not 3000)

```tsx
// extra-holidays.tsx
const config = {
  name: 'Extra holidays',
  icon: '🌴',
  flowType: 'choice',
  budgetPots: ['bonus', 'eoy'],
  dayValue: 180,
  maxDays: 5,
};

export function ExtraHolidaysFlow() {
  const [step, setStep] = useState(0);
  const [days, setDays] = useState(3);
  const [primary, setPrimary] = useState(null);
  const [tcsAccepted, setTcsAccepted] = useState(false);
  const totalCost = days * config.dayValue;

  return (
    <FlowFrame>
      {step === 0 && <IntroStep config={config} onStart={() => setStep(1)} />}
      {step === 1 && (
        <ConfigureStep
          config={config}
          input={
            <>
              <NumberStepper value={days} onChange={setDays} min={1} max={config.maxDays} />
              <BudgetSelector pots={config.budgetPots} selected={primary} onSelect={setPrimary} totalCost={totalCost} />
            </>
          }
          preview={
            <LivePreview
              label="Your choice"
              value={`${days} day${days === 1 ? '' : 's'}`}
              sub={`€${totalCost} total`}
            />
          }
          onContinue={() => setStep(2)}
          onBack={() => setStep(0)}
        />
      )}
      {step === 2 && (
        <ReviewStep
          config={config}
          summary={[
            { label: 'Days off', value: `${days} day${days === 1 ? '' : 's'}` },
            { label: 'Funded from', value: sourceLabel(primary) },
          ]}
          tcsAccepted={tcsAccepted}
          onTcsToggle={setTcsAccepted}
          onSubmit={() => setStep(3)}
          onBack={() => setStep(1)}
        />
      )}
      {step === 3 && (
        <SuccessScreen
          title={`${config.name} submitted!`}
          subtitle={`${days} days · €${totalCost} from ${sourceLabel(primary)} · awaiting HR`}
          timeline={CHOICE_FLOW_TIMELINE}
          primaryAction={{ label: 'View your extra holidays', onClick: () => setStep(4) }}
          secondaryAction={{ label: 'Back to benefits', onClick: () => onClose() }}
        />
      )}
    </FlowFrame>
  );
}
```

That's ~80 lines of meaningful logic. Compare to ~800 lines of HTML re-implementation. Component visuals come from Storybook automatically.

### What Claude should NOT do in Storybook composition mode

- Re-implement Button, Pill, Modal, Checkbox, Stepper, etc. — import them
- Hardcode color values — use tokens (`tokens.color.primary`, etc.)
- Build a custom radio when RichRadio exists
- Add CSS overrides via `style={{ ... }}` — extend the component if needed, don't patch
- Write `.css` blocks — components own their styling

### What Claude SHOULD do

- Compose existing components into the 4-step flow
- Wire state (which step, which input values, which budget)
- Pass benefit-specific props (icon, copy, math constants)
- Provide the per-benefit `config` object

### Pre-ship visual conformance check (HTML mode only)

When in HTML throwaway mode, before declaring done, Claude self-checks against pension savings as the canonical reference:

- [ ] Side panel width is exactly 416px on desktop
- [ ] Card padding is 24px (`p-6`)
- [ ] BenefitHeader icon is 56×56px (`w-14 h-14`)
- [ ] Section headings prefixed with ✦ glyph in `--color-brand-text` (NOT `--color-primary`)
- [ ] Mobile outer gutter is exactly 20px
- [ ] LivePreview / side panel sticks at `top:80px`
- [ ] Submit always enabled, inline error on click if T&C unchecked
- [ ] Bottom bar shows value summary, not instructions
- [ ] Universal "Submit choice" CTA + "[Benefit name] submitted!" success title
- [ ] Modals on mobile slide up as bottom drawers, full width, with handle indicator
- [ ] Stepper full-bleed on mobile, bordered on desktop
- [ ] Primary pink only on interactive elements

Conformance failures are the most likely place new prototypes drift. If 2+ items fail, treat as drift signal — the prototype isn't reusing patterns, it's inventing them.
