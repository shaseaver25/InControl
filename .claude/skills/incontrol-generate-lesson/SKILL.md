---
name: incontrol-generate-lesson
description: >
  Generates a complete 8-component InControl lesson as validated JSON, ready to
  upsert directly into Supabase lesson_steps. Use this skill whenever Shannon
  (or Nick) provides a lesson brief — title, skill area, learning objective,
  concept nodes, learner goal, daily routines, or any combination of those.
  Trigger on phrases like "generate a lesson", "write the lesson for", "create
  lesson content for", "build a lesson about", or "draft the [topic] lesson".
  Also trigger when someone pastes a brief and asks Claude to produce lesson
  content. The skill encodes all IDD/ASD accessibility rules and the exact JSON
  schema so callers never need to repeat them — token efficiency is the point.
---

## What this skill does

Takes a lesson brief → calls itself as the system prompt → outputs 8 `lesson_steps` rows as JSON → ready to upsert to Supabase.

When invoked: read the brief from the user's message (or ask for missing required fields), then generate the full lesson JSON following the rules and schema below. Output ONLY the JSON array — no preamble, no explanation after.

---

## Required brief fields (ask if missing)

| Field | Used in steps |
|---|---|
| `title` | All steps (context) |
| `skill_area` | All |
| `domain` | Metadata only |
| `learning_objective` | Steps 3, 4, 5 (what learner must do) |
| `real_world_anchor` | Steps 6, 7 |
| `learner_goal` | Step 2 (their long-term goal in their words) |
| `daily_routines` | Step 7 (list of existing routines) |

Optional but improves output: `special_notes` (constraints, sensitivities), `concept_nodes` (from learning graph), `month` (Intro/M1/M2/M3).

---

## Accessibility rules — apply to every step

These rules exist because the learner population (teens and adults with IDD/ASD) has specific, evidence-based needs. Violating them breaks the product.

1. **Grade 3 reading level.** Short sentences. No idioms. No abstract verbs ("utilize", "facilitate"). Active voice.
2. **Structured choice only.** Every choice screen: exactly 2–4 options. All options must be genuinely valid — no trick answers, no obviously wrong options included to test the learner. Every choice gets an emoji + a short label (≤3 words).
3. **Errorless learning on guided practice.** The hint must be written so that if the learner reads it, they will know the answer. It is shown *before* they tap anything — not after failure.
4. **Specific feedback, not generic.** "You remembered that water helps muscles work!" not "Great job!" Name the behavior, not the outcome.
5. **Never use the word "wrong."** Guidance feedback redirects: "Water is the one — tap the water glass!" Not "That's wrong, try again."
6. **Streaks are additive.** Never reference a "streak." Use "X times this month" framing only.
7. **Commitment anchor must exist.** The `anchor` in step 7 must be one of the routines from `daily_routines`. Do not invent one.

---

## Output schema

Return a JSON array of exactly 8 objects. Each object:

```json
{
  "step_number": <1–8>,
  "step_type": "<see below>",
  "content": { <step-specific fields> }
}
```

### Step 1 — `opener`
```json
{
  "greeting": "Hi, [Name]! <one warm sentence about today>",
  "mood_check": {
    "scale": "5-emoji",
    "options": ["😔","😐","🙂","😄","🤩"]
  }
}
```
Note: use `[Name]` as a literal placeholder — the app substitutes the learner's name at render time.

### Step 2 — `goal_connect`
```json
{
  "connection_sentence": "<one sentence linking today's skill to their long-term goal>",
  "progress_note": "<encouraging note about where they are in the series>"
}
```

### Step 3 — `teach`
```json
{
  "headline": "<question or statement, ≤8 words>",
  "key_points": [
    { "emoji": "🥤", "text": "<short factual point, ≤10 words>" },
    { "emoji": "⏰", "text": "<short factual point, ≤10 words>" },
    { "emoji": "💪", "text": "<short factual point, ≤10 words>" }
  ],
  "video_brief": "<2–3 sentences for video production team: what to show, what Coach Maya says, duration>"
}
```

### Step 4 — `guided_practice`
```json
{
  "question": "<single question, ≤12 words>",
  "choices": [
    { "emoji": "🥤", "label": "Water" },
    { "emoji": "🍬", "label": "Candy" },
    { "emoji": "🧃", "label": "Juice" },
    { "emoji": "☕", "label": "Coffee" }
  ],
  "correct_answer_label": "Water",
  "hint": "<sentence that makes the correct answer obvious without naming it>",
  "success_feedback": "<specific praise naming the behavior, ≤15 words>",
  "guidance_feedback": "<redirect to correct answer, no 'wrong', ≤12 words>"
}
```
Rules: 2–4 choices. All choices must be plausible real-world items in the same category. No trick answers.

### Step 5 — `independent`
```json
{
  "question": "<different question testing the same concept, ≤12 words>",
  "choices": [
    { "emoji": "🌅", "label": "Morning" },
    { "emoji": "🌙", "label": "Bedtime" },
    { "emoji": "🎬", "label": "Movies" },
    { "emoji": "🤷", "label": "Never" }
  ],
  "correct_answer_label": "Morning",
  "success_feedback": "<specific praise, ≤15 words>",
  "guidance_feedback": "<redirect, no 'wrong', ≤12 words>"
}
```
No hint field — this is the independent attempt. Question must test a different angle of the same learning objective than step 4.

### Step 6 — `choice_point`
```json
{
  "prompt": "<question asking how they'll apply the skill today, ≤10 words>",
  "choices": [
    { "emoji": "🥤", "label": "Water bottle by bed", "description": "See it right when you wake up" },
    { "emoji": "⏰", "label": "Phone reminder", "description": "Gets a notification" },
    { "emoji": "📝", "label": "Sticky note", "description": "See it while getting ready" },
    { "emoji": "🍳", "label": "While breakfast cooks", "description": "Already part of your morning" }
  ]
}
```
Rules: 2–4 choices. All choices must be genuinely useful real-world strategies. No "best" option — all are valid.

### Step 7 — `commitment`
```json
{
  "anchor": "<exact text of the daily routine from the brief>",
  "action": "<tiny, specific behavior — small enough to be 95%+ certain>",
  "anchor_emoji": "🍳",
  "action_emoji": "🥤",
  "reminder_time": "07:30"
}
```
The `anchor` must match one of the `daily_routines` from the brief verbatim. The `action` must be the smallest possible behavior that counts (not "drink more water" — "fill my water bottle and drink one glass").

### Step 8 — `celebration`
```json
{
  "praise": "<specific praise referencing today's lesson content, ≤20 words>",
  "progress_note": "<how many lessons done, what's next>",
  "next_lesson_preview": "<title of conceptually next lesson in the series>"
}
```
Praise must reference something specific from this lesson, not be generic ("You finished!").

---

## Usage in Shannon's Lovable/React app

Read `references/system-prompt.md` for the exact system prompt to paste into your Anthropic SDK call. The schema above is the contract — the system prompt enforces it on Claude's side.

**Supabase upsert pattern (TypeScript):**
```typescript
const steps = JSON.parse(claudeResponse); // the 8-item array

const { error } = await supabase
  .from('lesson_steps')
  .upsert(
    steps.map((step: LessonStep) => ({
      lesson_id: lessonId,
      step_number: step.step_number,
      step_type: step.step_type,
      content: step.content,
    })),
    { onConflict: 'lesson_id,step_number' }
  );
```

**Token budget:** A well-formed brief + system prompt + generation runs ~2,500–3,500 output tokens. Use `claude-3-5-haiku` for generation (fast, cheap, sufficient for structured JSON). Reserve Sonnet for cases where Nick flags a step for regeneration with complex feedback.

---

## When to ask vs. generate

- **Missing `learner_goal` or `learning_objective`**: ask — these shape steps 2, 4, 5 deeply.
- **Missing `daily_routines`**: ask — step 7 cannot be generated safely without them.
- **Missing `special_notes`**: skip, proceed.
- **Missing `domain` or `month`**: skip, proceed (metadata only, doesn't affect content).
- **Ambiguous anchor**: if multiple routines could work for step 7, pick the most morning-adjacent one — habits anchor best to the first routine of the day.
