# InControl — Frontend Design Plan

**Prepared for:** Shannon Seaver  
**Date:** June 2026  
**Stack:** Lovable (React/Vite) + Supabase  
**Artifacts in this folder:** design system · lesson player prototype · lesson builder prototype

---

## Decision-first summary

**Two surfaces, one design system, zero generic AI aesthetics.**

- **End-user app** — "Warm Intelligence" aesthetic. Cream + forest green + amber. Fredoka + Nunito. Every screen enforces the eight-component lesson structure with accessibility-first interaction patterns.
- **Lesson builder** — same tokens, dark sidebar, professional editor layout. Kelsey/Nick fill content; the AI pre-fills and they approve; the live preview shows exactly what learners see.

Both surfaces share one CSS variable set and one component library. Build the design system once, deploy twice.

---

## Aesthetic direction: Warm Intelligence

**Not:** Clinical healthcare blue. Purple-gradient AI slop. Condescending "kids app" primary colors. Generic Tailwind defaults.

**Yes:** A well-designed nature center for adults. Confident, warm, organized. Feels intentional for this specific population — not a reskin of something else.

| Token | Value | Role |
|---|---|---|
| Cream `#F7F3EC` | Background | Warm, easy on eyes, not stark white |
| Forest `#2E6844` | Primary | Trust, growth, calm — the spine of the UI |
| Amber `#E07C30` | Accent | Energy, celebration, CTA — use sparingly |
| Mint `#3DB89A` | Success | Achievement, positive feedback only |
| Sky `#4A90C4` | Info/hint | Guidance, prompts — never error |
| Ink `#1A1714` | Text | Warm black, never pure #000 |

**Fonts:**
- `Fredoka` — display, headings, button labels, coach name, step labels. Rounded, warm, accessible, distinctive. Not a common AI-slop choice.
- `Nunito` — body copy, instructions, feedback text. Highly legible, friendly weight range.

---

## Surface 1: End-User Lesson Player

Open [`incontrol-lesson-player.html`](./incontrol-lesson-player.html) to see the interactive prototype (click through all 8 screens).

### Structure

Every lesson is exactly 8 screens, always in this order. The structure does not change — only the content inside each screen varies.

| # | Screen | Key interaction | Design constraint |
|---|---|---|---|
| 1 | **Opener** | Mood check (5 emoji faces) | Same greeting, same coach, every time. Never skip. |
| 2 | **Goal connect** | Static (read/listen) | Shows user's own goal. Must reference their words. |
| 3 | **Teach** | Video player + 3 key points | Video required. Captions on by default. Audio narration on. |
| 4 | **Guided practice** | 2–4 choice cards, hint shown first | Errorless: hint appears *before* any tap. Never punish wrong answer. |
| 5 | **Independent** | 2–4 choice cards, no hint | Immediate ✅ feedback under 2 seconds. Specific praise, not generic. |
| 6 | **Choice point** | 2–4 big choice cards | All options valid. No trick. Visual + word for each. |
| 7 | **Tiny commitment** | Confirm the B=MAP action | Anchor → behavior shown visually with emoji chain. |
| 8 | **Celebration** | Animation + progress bar | Consistent every time. Shows next lesson. Progress visible. |

### Always-visible elements

**Visual schedule (top bar):** All 8 steps shown as icons across the top — greyed for future, filled for past, amber-highlighted for current. Never hides. Reduces anxiety, sets expectations, satisfies UDL representation.

**Coach chip:** Same avatar + name on every screen. Continuity = trust.

**Audio button:** Every screen has a 🔊 button. Tapping reads the screen aloud. Not optional — required for low-literacy users.

### Interaction rules (non-negotiable)

1. **One question per screen.** Never stack two decisions.
2. **Structured choice only.** 2–4 options with icon + word. No open text input.
3. **Hint before failure.** On guided practice, the hint chip shows before the user taps anything wrong. On independent, it appears after one wrong tap.
4. **Immediate feedback.** Under 2 seconds. Visual (color change, checkmark) + auditory (sound + narration) together.
5. **No red ❌ / no "wrong".** Wrong answers show amber guidance, then show the correct answer highlighted. Never say "wrong," "incorrect," or "try again."
6. **Streak is additive.** Show "12 times this month" — not "12-day streak." Streaks break. Monthly counts don't.
7. **Back button always available.** Users can go back. No forced linear progression without escape.
8. **Touch targets ≥ 56px.** Every tappable element. No exceptions.

### Component library (Lovable components to build)

| Component | Description |
|---|---|
| `<LessonShell>` | Phone frame, status bar, coach chip, visual schedule, bottom bar |
| `<VisualSchedule>` | 8-step icon row, done/active/future states |
| `<MoodCheck>` | 5 emoji buttons, tap to select, auto-advance |
| `<GoalCard>` | Forest green card with user's goal text + photo |
| `<VideoPlayer>` | Video with captions on, narration toggle, CC overlay |
| `<ChoiceGrid>` | 2 or 4 choice cards, icon + label, selection state |
| `<HintChip>` | Sky blue pill that appears before/after failure |
| `<FeedbackBanner>` | Mint ✅ success or amber 💡 guidance — never ❌ |
| `<CommitmentCard>` | "After I [anchor], I will [action]" + emoji chain |
| `<CelebrationScreen>` | Confetti overlay, progress bar, next lesson preview |
| `<ProgressBar>` | Forest green fill, 🌿 emoji at tip, percentage label |
| `<AudioButton>` | Global 🔊 button, triggers TTS for current screen |

---

## Surface 2: Lesson Builder (Kelsey/Nick)

Open [`incontrol-lesson-builder.html`](./incontrol-lesson-builder.html) to see the builder prototype.

### Layout: 3-column

```
[Sidebar 240px] [Editor flex] [Preview 320px]
```

- **Sidebar:** Dark (`#1E2A22`), lists all 8 steps + lesson settings. Done/active/pending states. Click to jump.
- **Editor:** Main content area. One step at a time. Field groups with labels, constraints, char counts.
- **Preview panel:** Live mini-phone showing exactly what the learner sees. Updates as you type.

### Editor flow per step

1. **AI generates** a draft (question + choices) from the Step 3 "Teach" content — one click.
2. **Nick edits** — changes wording, swaps emoji, reorders choices.
3. **Accessibility check panel** validates in real time (choice count, hint presence, emoji set, reading level).
4. **Kelsey approves** — moves step to "done," unlocks next step.

### Key builder constraints enforced by the UI

| Rule | How the builder enforces it |
|---|---|
| Max 4 choices | "Add option" button disabled after 4 |
| No trick answers | Field label reads "All must be valid choices · No trick answers" |
| Hint required | Required field — can't mark step done without it |
| Success feedback must be specific | Placeholder shows example: "You remembered to check the label for sugar" |
| Guidance feedback can't say "wrong" | Validation check flags the word "wrong" in feedback text |
| Reading level | Real-time Flesch-Kincaid estimate shown; target is Grade 3 |

### Builder component library (Lovable)

| Component | Description |
|---|---|
| `<BuilderShell>` | Top nav, sidebar, editor, preview layout |
| `<StepSidebar>` | 8 step items + settings, done/active/pending states |
| `<StepEditor>` | Switches content based on current step |
| `<AIGenerateBanner>` | One-click AI draft for any step |
| `<ChoiceBuilder>` | Emoji picker + text input rows, drag to reorder |
| `<MiniPhonePreview>` | Scaled-down lesson player preview, live-updating |
| `<AccessibilityChecker>` | Real-time validation panel, green ✅ / amber ⚠️ |
| `<LessonProgressBar>` | Top bar showing N/8 steps complete |
| `<StatusChip>` | Draft / In review / Approved / Published |

---

## Supabase schema (aligned to design)

These tables match what the lesson player and builder need. Shannon refines — this is the conceptual shape.

```sql
-- Users (learners)
users (id, name, avatar_url, created_at)

-- Learner profiles (set during onboarding)
learner_profiles (
  id, user_id,
  long_term_goal text,
  goal_photo_url text,
  daily_routines jsonb,        -- [{label, time, emoji}]
  modality_prefs jsonb,        -- {audio: true, video: true, captions: true}
  caregiver_contact jsonb,
  mood_baseline int            -- 1-5
)

-- Lessons (content, authored in builder)
lessons (
  id, title, skill_area, domain,
  month text,                  -- Intro/M1/M2/M3
  status text,                 -- draft/review/approved/published
  authored_by uuid,            -- Nick's user id
  approved_by uuid,            -- Kelsey's user id
  created_at, updated_at
)

-- Lesson steps (8 per lesson, ordered)
lesson_steps (
  id, lesson_id,
  step_number int,             -- 1-8
  step_type text,              -- opener/goal/teach/practice/independent/choice/commit/celebrate
  content jsonb,               -- all step-specific fields
  audio_url text,              -- generated TTS
  video_url text,
  created_at, updated_at
)

-- Learner completions
lesson_completions (
  id, user_id, lesson_id,
  completed_at timestamptz,
  mood_at_start int,           -- 1-5
  steps_completed int,         -- how far they got
  choice_made text,            -- screen 6 selection
  commitment_text text,        -- screen 7 confirmed action
  anchor_routine text          -- which routine they anchored to
)

-- Tiny commitments (tracked separately for reminder system)
commitments (
  id, user_id, lesson_id,
  anchor_routine text,
  action_text text,
  reminder_time time,
  created_at,
  completed_dates date[]       -- additive log, never streak
)

-- Concept mastery (L3 learning graph layer)
concept_mastery (
  id, user_id, concept_id,
  status text,                 -- not-ready/ready/practicing/mastered
  last_updated timestamptz
)
```

---

## Build order

Build in this order. Each phase is a shippable slice.

### Phase 1 — Design system + shell (Week 1)
- CSS variables file (`tokens.css`)
- `<LessonShell>` component with visual schedule, coach chip, audio button, bottom bar
- `<ChoiceGrid>` component — the most-used learner interaction
- `<MoodCheck>` component
- Storybook or Lovable component playground

### Phase 2 — Lesson player (Weeks 2–3)
- All 8 screen components wired together
- Navigation (next/back), visual schedule updates
- Static content (hardcode one full lesson)
- Audio button → browser TTS as placeholder
- Supabase: `lessons`, `lesson_steps`, `lesson_completions` tables

### Phase 3 — Onboarding (Week 3)
- Onboarding flow: name, long-term goal (picture support), 2–3 daily routines, modality preferences
- Supabase: `learner_profiles` table
- Goal card and commitment anchor pull from profile

### Phase 4 — Tiny commitment + reminder loop (Week 4)
- Commitment card with anchor selector (from learner's saved routines)
- Push notification scheduling
- Completion logging to `commitments` table
- Progress visualization (home screen)

### Phase 5 — Lesson builder (Weeks 5–6)
- Builder shell (3-column layout)
- Step editor for all 8 step types
- Live mini-phone preview
- Accessibility checker
- Supabase: lesson authoring, status workflow
- AI generation: Claude API call with concept graph context

### Phase 6 — AI generation (Week 7)
- Connect lesson builder's "Generate with AI" to Claude API
- Prompt includes: concept from learning graph, learner profile shape, step type, reading level target
- Output: structured JSON matching `lesson_steps.content` schema
- Nick reviews + edits, Kelsey approves

---

## What makes this unforgettable

Three things, executed precisely:

1. **The visual schedule never disappears.** Most accessible apps hide progress to save space. InControl keeps it visible always — it's the anxiety-reducing mechanism, not a nav element. This is the most visible embodiment of the design philosophy.

2. **Amber is earned.** Forest green is the UI's resting state. Amber only appears at moments of energy and celebration (CTA buttons, the active schedule step, achievement moments). When learners see amber, something important is happening.

3. **The builder previews in a phone.** Kelsey and Nick never guess what learners will see. The 320px live mini-phone on the right updates as they type. The constraint chip ("max 4 choices") is built into the field label, not a modal warning. Constraints are visible at authoring time, not publishing time.

---

## Files in this folder

| File | Purpose |
|---|---|
| `incontrol-design-system.html` | Full design system: colors, type, components, accessibility rules |
| `incontrol-lesson-player.html` | Interactive end-user lesson player (all 8 screens, clickable) |
| `incontrol-lesson-builder.html` | Lesson builder tool (Kelsey/Nick content creator surface) |
| `FRONTEND_DESIGN_PLAN.md` | This document |

All HTML files are self-contained — open in any browser, no server required.
