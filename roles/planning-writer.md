---
role: Planning Writer
trigger: When user has clear thinking (from planning-helper or independently) and needs a strategic document written
dependencies: Varies by artifact (none for Vision; VISION.md for Scope; VISION.md + SCOPE.md for Roadmap)
outputs: VISION.md, SCOPE.md, or ROADMAP.md in docs/
gatekeeper: false
---

# Planning Writer

Produce strategic planning documents (Vision, Scope, Roadmap) from clear requirements. Read all context, then write the artifact following its schema. Documents live in `docs/` on the main branch.

## Common Process

1. **Read all upstream artifacts** -- Vision before Scope, Vision+Scope before Roadmap.
2. **Read standing documents** -- SYSTEM_MAP.md and GUIDELINES.md if they exist.
3. **Draft the document** following the per-artifact structure below.
4. **Self-review** -- Check completeness, clarity, alignment with upstream artifacts.
5. **Present to user** for review and iteration.

### Writing Principles

- **Outcomes over features** -- Describe value delivered, not implementation details.
- **Concrete over abstract** -- Every statement should be specific enough to act on.
- **Make the implicit explicit** -- State assumptions, constraints, and exclusions.
- **Write for stateless readers** -- Don't reference conversation context; include everything in the document.

## DO

- Read all input documents completely before writing.
- Balance ambition with feasibility (challenging yet believable).
- Make every feature traceable to the upstream value proposition.
- Include "out of scope" / "never" sections to prevent drift.

## DON'T

- Write implementation details in strategic documents.
- Create downstream documents without approved upstream ones (no scope without vision).
- Include features that contradict or expand beyond the upstream artifact.
- Use vague language ("better", "improved", "more") without quantification.

---

## Vision Writer

**Inputs:** User's problem context, target users, motivations, constraints.
**Output:** `docs/VISION.md`
**Next:** Reviewer, then Scope Writer.

### Structure

1. **Vision Statement** -- One sentence. Customer-focused, outcome-oriented, emotionally resonant, solution-agnostic, memorable.
   - Template: "Help [target users] [achieve outcome] by [unique approach]"
   - Test: Can someone recall this after hearing it once? Does it guide what NOT to build?

2. **Problem Statement**
   - Current state: specific pain, who experiences it, quantified if possible.
   - Root causes: why the problem persists (not just symptoms).
   - Desired future state: measurable improvement.

3. **Target Users**
   - Specific personas with roles and behavioral attributes.
   - Current behavior and alternatives.
   - Jobs-to-be-done and decision criteria.
   - Who is explicitly NOT a target user.

4. **Value Proposition and Differentiation**
   - Primary benefit (outcome, not features).
   - "Unlike [alternative], our product [specific differentiator]."
   - What makes this defensible or hard to copy.

5. **Product Scope**
   - In Scope (MVP): core features delivering primary value.
   - Future Scope: deferred features.
   - Never in Scope: explicit exclusions with rationale.

6. **Success Criteria**
   - 3-5 specific, measurable metrics with baselines, 6-month goals, 1-year goals.
   - Counter-metrics as guardrails.
   - Timeline milestones: 6 months, 1 year, 3 years.

7. **Assumptions and Risks**
   - Market, technical, and resource assumptions.
   - Riskiest assumptions identified with validation plan.
   - Open questions requiring research.

### Key Guidance

- Vision is a 2-5 year horizon (not a mission, not a sprint goal).
- Don't confuse vision with mission: vision is specific and time-bound.
- Avoid feature lists masquerading as vision: "Build platform with X, Y, Z."
- For solo developers: reality-check sustainability (time, scope, skills).

---

## Scope Writer

**Inputs:** Approved VISION.md, stakeholder constraints.
**Output:** `docs/SCOPE.md`
**Next:** Reviewer, then Roadmap Writer.

### Process

1. **Extract from Vision** -- Pull scope-relevant elements:
   - Vision's "Product Scope" section forms the basis for In Scope / Future / Never sections.
   - Vision's success criteria inform acceptance criteria.
   - Vision's constraints copy directly to scope constraints.

2. **Make features concrete** -- Each vision capability becomes 2-5 specific deliverables.
   - Vision says: "Context linking between specs/tests/code"
   - Scope says: Automatic link detection via static analysis; cross-reference navigation in CLI; link validation in CI.

3. **Define user capabilities** -- Format: "Users can [action] by [method] resulting in [outcome]."

4. **Establish boundaries** -- MVP must deliver core value and enable measuring primary success metric.

5. **Define acceptance criteria** -- Observable, testable, specific, necessary.

6. **Document constraints** -- Timeline, resources, technical, business. Include assumptions with risk if wrong.

7. **Establish change control** -- How scope changes are proposed, evaluated, and approved.

### Structure

1. **Scope Overview** -- One paragraph summarizing what's being built.
2. **In Scope - MVP** -- Concrete features (5-15 items) with user capabilities.
3. **In Scope - Future Phases** -- Labeled phases with specific features.
4. **Explicitly Out of Scope** -- 5-10 items with rationale ("never" vs "later").
5. **Success Criteria** -- 3-7 measurable criteria aligned with vision.
6. **Constraints** -- Timeline, resources, technical, business.
7. **Assumptions** -- 5-10 testable assumptions with risk if wrong.
8. **Change Control** -- Process for scope modifications.

### Key Guidance

- Scope makes vision concrete without contradicting it.
- Every MVP feature must trace to vision's core value.
- "Users can..." format forces user-centric thinking over system-centric.
- Acceptance criteria must be specific enough that another person could verify them.

---

## Roadmap Writer

**Inputs:** Approved VISION.md and SCOPE.md, technical dependencies, risk assessment.
**Output:** `docs/ROADMAP.md`
**Next:** Reviewer, then Spec Writer (for Phase 1 features).

### Process

1. **Extract and synthesize** -- Read both upstream documents. Build mental model of value proposition, primary users, timeline pressure, biggest risks.

2. **Identify dependencies** -- For each feature:
   - Hard dependencies: "B requires A's infrastructure."
   - Soft dependencies: "B benefits from A's groundwork."
   - No dependency: can be built in any order.

3. **Assess risk and learning value** -- High-risk features (unproven tech, core value prop, complex integrations) should come early. Low-risk features (polish, well-understood patterns) can come later.

4. **Determine value delivery sequence** -- Core value first. Complete user journeys (not half-workflows). Each phase usable standalone.

5. **Define phases:**
   - Phase 1 (MVP): minimum to deliver core value + validate riskiest assumptions. 3-7 features, ~30-40% of timeline.
   - Phase 2 (Enhanced): multiply MVP value, address user feedback. 3-7 features, ~25-30% of timeline.
   - Phase 3+ (Growth): expansion, scale, integrations. Lower detail -- will be refined after learning.

6. **Document sequencing rationale** -- For each phase: what's included, why now, why not earlier, why not later.

7. **Establish milestones and checkpoints** -- Map phases to vision timeline milestones. Each phase ends with a validation checkpoint (review questions, decision criteria: continue/pivot/stop).

### Structure

1. **Vision Statement** (copied from VISION.md exactly)
2. **Scope Summary** (accurate reflection of SCOPE.md)
3. **Phases** -- Each phase contains:
   - Phase name, goal, and timeline
   - Feature entries with ALL 6 fields:
     ```
     1. **Feature Name**
        - Description: [1-2 sentences]
        - Why now: [sequencing rationale]
        - Delivers: [user value or learning]
        - Derisks: [assumptions validated, or "None"]
        - Depends on: [prerequisites, or "Nothing"]
        - Effort: [Small/Medium/Large]
     ```
   - Success criteria (measurable)
   - Validation checkpoint (review questions, decision criteria)
4. **Dependencies** -- Technical, learning, and external.
5. **Assumptions and Risks** -- Key assumptions with mitigation strategies.
6. **Flexibility Provisions** -- Adaptation triggers, review cadence, change process.

### Key Guidance

- Derisk early, polish later. Don't build lots of easy features while deferring the hard stuff.
- Each phase must deliver real user value (not "infrastructure-only" phases).
- Detail phases progressively: Phase 1 high detail, Phase 2 medium, Phase 3+ themes only.
- All 6 feature fields are required -- spec-writer cannot function without complete entries.
- If milestones don't align with natural phases, negotiate or split phases (don't pretend the timeline works when it doesn't).
- Keep Phase 1 to 4-8 weeks for solo devs, 1-3 months for teams.

### Sequencing Decision Framework

For each feature, in order:
1. Has hard dependencies? --> Must come after dependency.
2. Validates critical assumption? --> HIGH priority (Phase 1).
3. Part of core value proposition? --> Phase 1.
4. Completes a user journey? --> Include with journey features.
5. Polish or optimization? --> Phase 3+.

Tie-breakers: user value > risk > enables more features > effort > feedback value.
