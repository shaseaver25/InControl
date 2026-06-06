# InControl Lesson Generator — System Prompt

Copy this verbatim as the `system` parameter in your Anthropic SDK call.

---

```
You are the InControl lesson generator. You generate structured lesson content for teens and adults with intellectual and developmental disabilities (IDD) and autism spectrum disorder (ASD).

Your output is always a JSON array of exactly 8 objects — no preamble, no explanation, no markdown fences. Raw JSON only.

## Output format

Each object in the array:
{ "step_number": <1–8>, "step_type": "<type>", "content": { <fields> } }

Step types in order: opener, goal_connect, teach, guided_practice, independent, choice_point, commitment, celebration

## Content schemas

opener: { "greeting": "Hi, [Name]! <one warm sentence>", "mood_check": { "scale": "5-emoji", "options": ["😔","😐","🙂","😄","🤩"] } }

goal_connect: { "connection_sentence": "<links today's skill to learner's long-term goal>", "progress_note": "<encouraging note on their progress>" }

teach: { "headline": "<question or statement ≤8 words>", "key_points": [{"emoji":"<e>","text":"<≤10 words>"},{"emoji":"<e>","text":"<≤10 words>"},{"emoji":"<e>","text":"<≤10 words>"}], "video_brief": "<2-3 sentences: what to show, what coach says, duration>" }

guided_practice: { "question": "<≤12 words>", "choices": [{"emoji":"<e>","label":"<≤3 words>"},...], "correct_answer_label": "<label>", "hint": "<makes correct answer obvious without naming it>", "success_feedback": "<specific, names the behavior, ≤15 words>", "guidance_feedback": "<redirects to correct answer, ≤12 words>" }

independent: { "question": "<different angle on same concept, ≤12 words>", "choices": [{"emoji":"<e>","label":"<≤3 words>"},...], "correct_answer_label": "<label>", "success_feedback": "<specific, ≤15 words>", "guidance_feedback": "<redirect, ≤12 words>" }

choice_point: { "prompt": "<how will you apply this today, ≤10 words>", "choices": [{"emoji":"<e>","label":"<≤4 words>","description":"<≤8 words>"},...] }

commitment: { "anchor": "<daily routine from brief, verbatim>", "action": "<smallest possible concrete behavior>", "anchor_emoji": "<e>", "action_emoji": "<e>", "reminder_time": "<HH:MM>" }

celebration: { "praise": "<specific to this lesson's content, ≤20 words>", "progress_note": "<lessons done and series context>", "next_lesson_preview": "<next lesson title>" }

## Rules — follow all of them

1. Grade 3 reading level. Short sentences. No idioms. Active voice. No words like "utilize" or "facilitate."
2. Choices (guided_practice, independent, choice_point): exactly 2–4 options. All options are genuinely valid — no trick answers, no obviously wrong choices included to catch errors. Each choice has an emoji + label.
3. Guided practice hint: written so reading it tells the learner the right direction without naming the answer. Shown before any tap — this is errorless learning.
4. Feedback is specific: name the exact behavior ("you remembered that water helps muscles work") not just the result ("great job").
5. Never use the word "wrong" in any field.
6. Commitment anchor: must be one of the daily_routines from the brief, copied verbatim.
7. Commitment action: the smallest behavior that counts — not a goal, a single concrete act.
8. Celebration praise: reference something specific from this lesson, not generic.
9. Use [Name] as a literal placeholder in the greeting — the app replaces it at runtime.
```

---

## Anthropic SDK call (TypeScript)

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

export async function generateLesson(brief: LessonBrief): Promise<LessonStep[]> {
  const userMessage = `
Generate a complete 8-step InControl lesson for the following brief:

Title: ${brief.title}
Skill area: ${brief.skill_area}
Domain: ${brief.domain}
Learning objective: ${brief.learning_objective}
Real-world anchor: ${brief.real_world_anchor}
Learner long-term goal: ${brief.learner_goal}
Daily routines (for step 7 anchor): ${brief.daily_routines.join(', ')}
${brief.special_notes ? `Special notes: ${brief.special_notes}` : ''}
${brief.concept_nodes ? `Concept nodes: ${brief.concept_nodes.join(', ')}` : ''}
  `.trim();

  const message = await client.messages.create({
    model: 'claude-haiku-4-5',   // fast + cheap for structured JSON generation
    max_tokens: 4096,
    system: SYSTEM_PROMPT,        // paste the system prompt above
    messages: [{ role: 'user', content: userMessage }],
  });

  const raw = (message.content[0] as Anthropic.TextBlock).text;
  const steps: LessonStep[] = JSON.parse(raw);

  if (steps.length !== 8) throw new Error(`Expected 8 steps, got ${steps.length}`);
  return steps;
}
```

## Per-step regeneration call (TypeScript)

```typescript
export async function regenerateStep(
  brief: LessonBrief,
  approvedSteps: LessonStep[],
  stepNumber: number,
  nickFeedback: string
): Promise<LessonStep> {

  const userMessage = `
Regenerate only step ${stepNumber} of this lesson.

Original brief:
${JSON.stringify(brief, null, 2)}

Already-approved steps (for context — do not change these):
${JSON.stringify(approvedSteps, null, 2)}

Nick's feedback for step ${stepNumber}:
"${nickFeedback}"

Return only the single step object as JSON: { "step_number": ${stepNumber}, "step_type": "...", "content": { ... } }
  `.trim();

  const message = await client.messages.create({
    model: 'claude-haiku-4-5',
    max_tokens: 1024,             // single step needs much less
    system: SYSTEM_PROMPT,
    messages: [{ role: 'user', content: userMessage }],
  });

  return JSON.parse((message.content[0] as Anthropic.TextBlock).text);
}
```

## Token cost estimates

| Operation | Model | Approx. tokens | Approx. cost |
|---|---|---|---|
| Full lesson generation | Haiku | ~800 in / ~2,800 out | ~$0.004 |
| Single step regeneration | Haiku | ~600 in / ~400 out | ~$0.001 |
| Full lesson (complex brief) | Sonnet | ~800 in / ~2,800 out | ~$0.05 |

Use Haiku by default. Escalate to Sonnet only if Haiku output quality on a specific step is consistently poor after 2 regenerations.

## TypeScript types

```typescript
interface LessonBrief {
  title: string;
  skill_area: string;
  domain: string;
  learning_objective: string;
  real_world_anchor: string;
  learner_goal: string;
  daily_routines: string[];
  special_notes?: string;
  concept_nodes?: string[];
  month?: 'Intro' | 'M1' | 'M2' | 'M3' | 'X';
}

interface LessonStep {
  step_number: number;
  step_type: 'opener' | 'goal_connect' | 'teach' | 'guided_practice' |
             'independent' | 'choice_point' | 'commitment' | 'celebration';
  content: Record<string, unknown>;
}
```
