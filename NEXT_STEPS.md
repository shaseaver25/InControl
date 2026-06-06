# InControl — Next Steps Plan

**As of:** June 6, 2026  
**Shannon's build:** Lesson generator app (Lovable + Supabase + Claude API)  
**Other engineer's build:** Learner-facing app (reads from Supabase)

---

## Decisions to make first (blockers)

These will stall you if left open. Answer them before writing any code.

| Decision | Options | Recommendation |
|---|---|---|
| **Auth in the generator app** | No auth (URL-protected internal tool) vs. Supabase auth | Supabase auth — but Nick does NOT need his own Supabase account (see below) |
| **Streaming vs. batch generation** | Stream each step as it arrives vs. wait for all 8 then display | Streaming — the reveal is the hero moment; Lovable supports it via fetch + ReadableStream |
| **Concept picker data source** | Load nodes CSV into Supabase table vs. hardcode domains/months only | Seed into Supabase — unlocks prerequisite auto-population and future graph features |
| **Handoff timing with other engineer** | Share JSON schema now vs. wait until first lesson is published | Share now — they can build render components in parallel while you build the generator |

---

## Phase 1 — Foundation (do this before any UI work)

**Goal:** Supabase is ready, Lovable project exists, API key is set, first lesson can write to the database.

### 1a. Supabase setup
Run the DDL from `ABOUT.md` in the Supabase SQL editor:
- `lessons` table
- `lesson_steps` table (with unique constraint on `lesson_id + step_number` for safe upsert)

Then seed the concept graph:
```sql
-- Create a simple concepts table for the picker
create table concepts (
  id text primary key,          -- matches nodes CSV "id" column
  label text not null,
  domain text,
  role text,                    -- foundation / concept / skill / goal
  month text,
  modifiable boolean,
  description text
);
```
Import `incontrol_learning_graph_nodes.csv` → Supabase Table Editor → Import CSV.  
Same for edges (needed for prerequisite auto-population later).

### 1b. Add Nick as a user (not a Supabase account)

Nick does not need his own Supabase account. He needs a user account *in your app*, which lives in Supabase Auth — your project, not his.

Two ways to create it:
- **Dashboard:** Supabase → Authentication → Users → Invite user → `nick.ziebell@incontrolmn.com` → Send invite. He gets an email, sets a password, done.
- **In code:** `supabase.auth.admin.inviteUserByEmail('nick.ziebell@incontrolmn.com')` — call this from a one-time admin script or a simple invite page you build later.

His account is just a row in `auth.users`. That row ID is what populates `authored_by` on every lesson he generates. He never touches the Supabase dashboard.

### 1c. Lovable project
- Create new Lovable project, connect to your Supabase project
- Add environment variable: `ANTHROPIC_API_KEY`
- Confirm Supabase client is wired (it comes default in Lovable)

### 1d. Smoke test the API call
Before building any UI, paste the system prompt from `references/system-prompt.md` into a simple test function and call it with a hardcoded brief. Confirm you get valid JSON back. This is the riskiest integration — test it first.

**Done when:** You can call Claude API from Lovable, get 8-step JSON, and upsert it to `lesson_steps`.

---

## Phase 2 — Brief form + generation (core product)

**Goal:** Nick can fill out a brief, hit Generate, see 8 step cards, approve them, and publish to Supabase.

### Build order within this phase:
1. **Brief form** — all fields from `LESSON_GENERATOR_PLAN.md`. Start with just the required fields (title, objective, learner goal, daily routines). Add concept picker after the smoke test is passing.
2. **Generation screen** — 8 `StepCard` components, one per step type. Each renders the `content` JSON fields for its step type. Static first (hardcode a lesson JSON to render), then wire to the API.
3. **Approve + publish flow** — approve button per card, disabled Publish button until all 8 approved, upsert to Supabase on Publish.
4. **Library view** — table of lessons by status. Needed on day one so Nick can see what exists.

**Done when:** End-to-end works — brief → generate → approve → Supabase row with status `published`.

---

## Phase 3 — Edit loop

**Goal:** Nick can fix any step without regenerating the whole lesson.

1. **Inline editing** — make each `content` field editable (contenteditable or controlled inputs). Changes persist to `lesson_steps` on blur.
2. **Per-step regeneration** — "Regenerate this step" button opens a modal ("what should be different?"), calls Claude API with the approved steps as context + Nick's note, replaces that card's content.
3. **Unapprove** — let Nick un-approve a step if he edits it (resets its status to "needs review").

**Done when:** Nick can iterate on any step independently without losing other approved steps.

---

## Phase 4 — Polish + handoff prep

**Goal:** App is ready for Nick to use independently.

1. **Streaming reveal** — replace batch generation with streamed output. Each step card appears as Claude finishes it. Use `fetch` with `ReadableStream` and parse the JSON incrementally.
2. **Concept node picker** — searchable multi-select that pulls from the `concepts` Supabase table. Auto-populates prerequisites from the edges table when concepts are selected.
3. **Brief history** — store `brief_json` on the `lessons` row so any lesson can be re-opened and regenerated from its original inputs.
4. **Nick onboarding** — one-page walkthrough of the 4-step workflow (brief → generate → review → publish). Keep it short.

---

## Parallel track: other engineer handoff

Do this during Phase 1, not after Phase 4.

**Send them:**
- The JSON schema from `SKILL.md` (exact field names per step type)
- The Supabase query: `SELECT * FROM lesson_steps WHERE lesson_id = ? ORDER BY step_number`
- The `design/incontrol-lesson-player.html` prototype — this is what their app should render
- The `design/FRONTEND_DESIGN_PLAN.md` — design system tokens, component list, accessibility rules

**Agree on:**
- What `status` value triggers the lesson becoming visible in the learner app (`published`? a separate `live` flag?)
- Whether they need a webhook when a lesson is published or just polling

---

## What's already done

| Artifact | Status |
|---|---|
| Learning graph (133 nodes, 177 edges) | ✅ In repo |
| Course description | ✅ In repo |
| Frontend design system | ✅ `design/incontrol-design-system.html` |
| Learner app prototype (8 screens) | ✅ `design/incontrol-lesson-player.html` |
| Generator app prototype (3 views) | ✅ `design/incontrol-generator.html` |
| Generator app design plan | ✅ `design/LESSON_GENERATOR_PLAN.md` |
| Supabase schema (DDL) | ✅ In `ABOUT.md` |
| Claude skill (lesson generation) | ✅ Installed globally + in repo |
| System prompt + TypeScript SDK examples | ✅ `references/system-prompt.md` |
| GitHub repo | ✅ `shaseaver25/InControl` |

---

## Suggested order for your next session

1. Make the four decisions in the table at the top
2. Run the Supabase DDL (15 min)
3. Create the Lovable project + set env var (15 min)
4. Run the smoke test API call (30 min)
5. Build the brief form (2–3 hrs)

That's a full working day and it ends with the most critical integration proven out.
