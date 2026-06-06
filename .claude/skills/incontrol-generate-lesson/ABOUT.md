# incontrol-generate-lesson — About

What this skill needs to run, depending on who's using it and where.

---

## Two ways to use this skill

### 1. In Claude Code (Shannon or Nick generating lessons manually)

You type something like:

> "Generate a lesson for hydration — the learning objective is identifying water as the best drink for muscle health. Learner goal: feel stronger at the park. Daily routines: making breakfast, brushing teeth."

Claude reads the skill, generates the full 8-step JSON, and prints it in the chat. You copy it into Supabase or let the app upsert it.

**What you need:**
- Claude Code installed and running
- This skill installed at `~/.claude/skills/incontrol-generate-lesson/` (already done)
- Nothing else — no API keys, no dependencies

---

### 2. In the Lovable app (Nick clicks "Generate with AI" in the builder UI)

The app makes a direct call to the Anthropic API using the system prompt from `references/system-prompt.md`.

**What the app needs:**

| Requirement | Details |
|---|---|
| Anthropic API key | Add as `ANTHROPIC_API_KEY` in your Lovable environment variables (never hardcode) |
| `@anthropic-ai/sdk` npm package | `npm install @anthropic-ai/sdk` |
| Model access | `claude-haiku-4-5` — available on any paid Anthropic plan |
| Supabase client | Already in your stack — used to upsert the generated steps |

**What the app does NOT need:**
- This skill file (the skill is for Claude Code only)
- The `system-prompt.md` content lives in your app's source code, copied from `references/system-prompt.md`

---

## Environment variables

```bash
# .env (never commit this file)
ANTHROPIC_API_KEY=sk-ant-...       # Required for the Lovable app's API call
SUPABASE_URL=https://...           # Already in your stack
SUPABASE_ANON_KEY=...              # Already in your stack
```

In Lovable: Settings → Environment Variables → add `ANTHROPIC_API_KEY`.

---

## Supabase tables this skill writes to

The generated JSON maps directly to these two tables. They must exist before the app can publish a lesson.

```sql
-- Run once in Supabase SQL editor
create table if not exists lessons (
  id uuid primary key default gen_random_uuid(),
  title text not null,
  skill_area text,
  domain text,
  month text,
  status text default 'draft',  -- draft | review | approved | published
  authored_by uuid references auth.users(id),
  brief_json jsonb,              -- stores the original generation brief
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

create table if not exists lesson_steps (
  id uuid primary key default gen_random_uuid(),
  lesson_id uuid references lessons(id) on delete cascade,
  step_number int not null check (step_number between 1 and 8),
  step_type text not null,
  content jsonb not null,
  approved_by uuid references auth.users(id),
  approved_at timestamptz,
  unique (lesson_id, step_number)  -- enables upsert by lesson_id + step_number
);
```

---

## What the learner app (other engineer's build) needs

Nothing from this skill directly. It just reads from Supabase:

```sql
select * from lesson_steps
where lesson_id = '<id>'
order by step_number;
```

The `content` field is JSON — the other engineer renders each `step_type` using their own components. Share the JSON schema from `SKILL.md` with them so they know the exact field names for each step type.

---

## Cost

| What | Model | Per call |
|---|---|---|
| Full 8-step lesson | `claude-haiku-4-5` | ~$0.004 |
| Single step regeneration | `claude-haiku-4-5` | ~$0.001 |
| Full lesson (escalated) | `claude-sonnet-4-5` | ~$0.05 |

Use Haiku by default. Only escalate to Sonnet if Haiku output on a specific step is consistently off after two regeneration attempts with feedback.

---

## Files in this skill

| File | Purpose |
|---|---|
| `SKILL.md` | The skill itself — Claude Code reads this when you ask it to generate a lesson |
| `ABOUT.md` | This file — setup and dependencies |
| `references/system-prompt.md` | The system prompt to paste into your Lovable app's Anthropic SDK call, with TypeScript examples |
