# InControl — Lesson Generator App
## Shannon's Build · Frontend Design Plan

**What this is:** A Claude API-powered tool that takes a skill/topic brief and generates a complete 8-component lesson, then writes it to Supabase for the learner app to consume.

**Primary users:** Shannon (build + test), Nick (day-to-day authoring once handed off)

**Output:** Structured rows in Supabase `lessons` + `lesson_steps` tables

**Stack:** Lovable (React/Vite) + Supabase + Claude API (anthropic SDK)

---

## What the app actually does (the workflow)

```
Nick fills a brief (skill, domain, learner goal, concept from graph)
       ↓
Claude API generates all 8 lesson components (structured JSON)
       ↓
Nick reviews each component in a step-by-step editor
       ↓
Nick edits anything → re-generates individual steps if needed
       ↓
Nick approves → lesson writes to Supabase → learner app picks it up
```

This is **not** a general CMS. It's a generation workflow with a review gate before anything hits production.

---

## Three views, one app

### View 1 — Brief screen (the input)
Nick fills this before generation. The quality of the brief determines the quality of the output.

**Fields:**
| Field | Type | Notes |
|---|---|---|
| Lesson title | Text | "Drinking More Water" |
| Skill area | Text | "Hydration" |
| Domain | Select | 9 domains from learning graph |
| Month | Select | Intro / M1 / M2 / M3 |
| Concept node(s) | Multi-select | From `incontrol_learning_graph_nodes.csv` |
| Prerequisite concepts | Auto-populated | From graph edges — shown, not editable |
| Learning objective | Textarea | "Learner can identify water as the best choice for muscle health" |
| Real-world anchor | Text | "Filling a water bottle before leaving the house" |
| Long-term goal (learner) | Text | "Feel stronger and keep up with friends at the park" |
| Daily routine anchors | Tags | "Making breakfast," "Brushing teeth," etc. |
| Special notes | Textarea | "Avoid mentioning weight. Keep visuals food-positive." |

**Design:** One clean form. Generous spacing. Nick should be able to fill this in < 5 minutes.

---

### View 2 — Generation screen (the work)
After Nick hits "Generate Lesson," the app calls Claude API and streams the result.

**What happens:**
1. Spinner / streaming indicator per step as Claude generates
2. Each of the 8 components appears as it's generated (streamed, not all-at-once)
3. Each component card shows: the generated content + an "Edit" button + a "Regenerate this step" button
4. A global "Regenerate all" option

**The 8 component cards (what gets generated + what Nick sees):**

| Step | Claude generates | Nick reviews |
|---|---|---|
| 1 Opener | Coach greeting text, mood check config | Just approves (structure is fixed) |
| 2 Goal connect | One sentence connecting lesson → learner's goal | Edits wording if needed |
| 3 Teach | Lesson headline, 3 key points (icon + text each) | Confirms accuracy, swaps icons |
| 4 Guided practice | Question text, 4 choices (emoji + label), hint text, success/guidance feedback | Most critical review — check all choices are valid |
| 5 Independent | Question text, 4 choices, immediate feedback text | Same review as #4 |
| 6 Choice point | Prompt text, 4 application options (icon + label + 1-line description) | All options must be real and equal |
| 7 Tiny commitment | After I [anchor] → I will [action] (from daily routines + skill) | Check anchor exists in learner profile |
| 8 Celebration | Praise text (specific to today's lesson), next lesson preview title | Quick check |

**Design:** Vertical stack of cards. Each card has a header showing the step name + a status badge (Generated / Editing / Approved). Approved cards collapse. The "Approve all and publish" button at the bottom is disabled until all 8 are approved.

---

### View 3 — Lessons library
A table of all lessons with status (Draft / In Review / Published). Click any lesson to go back into its generation/review view.

| Column | Notes |
|---|---|
| Title | Clickable → opens lesson |
| Domain | Color-coded by the 9 domains |
| Month | Intro/M1/M2/M3 |
| Status | Draft → Approved → Published |
| Authored by | Nick / Shannon |
| Last updated | Relative time |

---

## Claude API prompt design

The system prompt needs to encode:
1. The 8-component lesson structure (from research doc)
2. The accessibility rules (structured choice, errorless, no trick answers, Grade 3 reading level)
3. The anti-patterns (streak shaming, generic praise, open text input)
4. The output schema (exact JSON shape matching Supabase `lesson_steps.content`)
5. The learning graph context (concept + prerequisites)

**Output format:** Claude returns a JSON array of 8 step objects. Each object matches the `lesson_steps.content` schema. This goes directly into Supabase — no transformation layer.

**Per-step regeneration:** When Nick hits "Regenerate this step," the API call includes:
- The original brief
- The already-approved steps (as context)
- Which step to regenerate
- Any note Nick typed ("Make the choices more concrete. Avoid abstract verbs.")

This is the most important UX decision: **Nick can give natural-language feedback on individual steps and get targeted regeneration** without blowing up the whole lesson.

---

## Supabase tables this app writes to

```
lessons (id, title, skill_area, domain, month, status, authored_by, brief_json, created_at, updated_at)
lesson_steps (id, lesson_id, step_number, step_type, content jsonb, approved_by, approved_at)
```

The `content` field is the JSON Claude generates. The learner app reads it and renders components from it. Shannon defines the schema; the other engineer reads it.

**`brief_json`** stores the full generation brief — so any lesson can be regenerated later from its original inputs.

---

## Aesthetic direction (same tokens, different feel)

Same design system (`tokens.css`) as the learner app, but:
- **More information-dense** — this is a work tool, not a gentle consumer app
- **Darker sidebar** (same `#1E2A22`) — professional context
- **Generation state is the hero moment** — the streaming cards appearing is the most important UX to get right. Each card should appear with a subtle fade-in as it populates.

**The one thing Nick will remember:** Watching the lesson appear piece by piece as Claude generates it. The streaming reveal should feel like a skilled collaborator building in front of you — not a loading spinner followed by a wall of text.

---

## Component list (Lovable)

| Component | Description |
|---|---|
| `<BriefForm>` | Full brief input, concept graph node picker, submit |
| `<ConceptPicker>` | Searchable multi-select from nodes CSV / Supabase |
| `<GenerationShell>` | 8-card vertical stack, global actions |
| `<StepCard>` | Individual step card: generated content + edit + regenerate + approve |
| `<StepContent_*>` | 8 variants matching each step type's content fields |
| `<StreamingIndicator>` | Per-card "Claude is writing..." animation |
| `<RegenerateModal>` | "What should be different?" textarea → targeted API call |
| `<LessonsTable>` | Library view, sortable, filterable by domain/status |
| `<StatusBadge>` | Draft / In review / Approved / Published pill |

---

## Build order for Shannon

**Week 1: Foundation**
- Supabase schema: `lessons` + `lesson_steps` tables
- Lovable project setup with `tokens.css` (from design system)
- Lessons library view (table, empty state)
- Brief form (all fields, validation)

**Week 2: Generation core**
- Claude API integration (Anthropic SDK)
- System prompt with lesson structure + accessibility rules + JSON schema
- Generation screen: all 8 `<StepCard>` variants rendering from JSON
- Write to Supabase on approval

**Week 3: Edit loop**
- Per-step edit (inline editing of generated content)
- Per-step regeneration with Nick's feedback note
- Approve-all → publish flow
- Status badge lifecycle

**Week 4: Polish + handoff**
- Streaming reveal animation (cards appear as Claude generates)
- Concept graph node picker (pull from `incontrol_learning_graph_nodes.csv` → Supabase seed)
- Prerequisite auto-population from edges CSV
- Brief → lesson history (can re-open and regenerate from original brief)
- Nick onboarding (simple walkthrough of the workflow)

---

## What to hand off to the other engineer

When a lesson status flips to `Published`:
- `lessons` row exists with full metadata
- `lesson_steps` rows exist (8 rows, ordered by `step_number`)
- Each `lesson_steps.content` follows the agreed JSON schema
- The learner app queries: `SELECT * FROM lesson_steps WHERE lesson_id = ? ORDER BY step_number`

The other engineer never touches the generator. Shannon owns the schema contract.
