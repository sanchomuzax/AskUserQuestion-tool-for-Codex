---
name: ask-user-question
description: "Interactive clarification helper that restates the current understanding, asks 1-5 targeted questions (multi-choice with Other) when requirements are missing, ambiguous, or preference-driven, then outputs normalized user_answers JSON and proceeds. Auto-trigger on unclear or multi-interpretation tasks or when explicitly told to 'kérdezz vissza', 'tisztázd', 'interjúzz meg', or similar requests to interview/clarify."
---

# Ask User Question

General-purpose interview flow that pauses to gather the minimum answers needed before continuing the original task.

## When to Trigger
- Ambiguity: multiple plausible interpretations, missing requirements, or blocked choices (e.g., platform, scope, format, priority).
- Preference or constraints needed: style, budget/time limits, risk tolerance, stack choice, access limits.
- Explicit cues: "kérdezz vissza", "tisztázd", "interjúzz meg", "ask user questions", "clarify first", "interview me".

## Core Workflow (follow in order)
1) **Restate (Step A):** Summarize current understanding in 2-4 sentences and say why clarification is needed.
2) **Ask (Step B):** Pose 1-5 questions only. Prefer multi-choice (2-6 options) plus an `Other (free text)` option. Each question must include:
   - `id` (stable key), `question` (concise), `why_it_matters` (1 short sentence), `options` (if applicable).
   Present them clearly under a "Questions" heading to signal pause.
3) **Pause (Step C):** Instruct the user to answer. Do not continue until answers arrive.
4) **Emit JSON (Step D):** After getting answers (or partial), output exactly one object:
   ```json
   {
     "user_answers": {
       "<key>": "...",
       "constraints": { "time": "...", "budget": "...", "platform": "..." },
       "preferences": { "style": "...", "risk": "..." }
     }
   }
   ```
   Normalize option labels; keep keys stable across rounds.
5) **Apply (Step E):** Continue the original task explicitly using the collected answers. If something is still missing, ask only the minimal follow-up.

## Question Design Rules
- Minimum viable set: ask only what blocks progress. Avoid more than 5 questions per round.
- Multi-choice first, but allow `Other (free text)` for flexibility.
- Scope the options to the current task; do not offer irrelevant stacks or formats.
- If the user explicitly asks for questioning, skip restatement brevity but still include Step A for traceability.

## Answer Handling
- Validate: If an answer is missing, contradictory, or out of scope, ask a single targeted follow-up for that part only.
- If the user declines to choose, pick a safe default and note it in the JSON.
- Keep the JSON stable and normalized even when partial; fill unknowns with clear placeholders like `"unknown"` rather than omitting keys.

## Hungarian Usage Examples
- "Kérlek, mielőtt nekilátsz, kérdezz vissza a hiányzó részletekre."
- "Ez többféleképp is érthető, tisztázd a preferenciáimat."
- "Interjúzz meg röviden, hogy tudd, mit szeretnék pontosan."

## Test Scenarios (for self-check)
1) **Feature ambiguity:** User: "Kell egy mobil app a futásaimhoz."  
   - Sample questions: platform (iOS/Android/both/Other), core goal (tracking/coach/share/Other), data sources (watch/phone/manual/Other).  
   - Expected JSON (example answers):  
     ```json
     {"user_answers":{"platform":"android","goal":"tracking","data_source":"watch","constraints":{"time":"unknown","budget":"unknown","platform":"android"},"preferences":{"style":"unknown","risk":"unknown"}}}
     ```
2) **Preference choice:** User: "Írj CI pipeline-t Node projekthez."  
   - Sample questions: CI provider (GitHub Actions/GitLab/Other), test scope (unit/unit+lint/full/Other), caching (yes/no/Other).  
   - Expected JSON (example answers):  
     ```json
     {"user_answers":{"ci_provider":"github_actions","test_scope":"unit+lint","caching":"yes","constraints":{"time":"unknown","budget":"unknown","platform":"unknown"},"preferences":{"style":"unknown","risk":"conservative"}}}
     ```
3) **Explicit cue:** User: "Kérdezz vissza: új landing page, modern vagy retro?"  
   - Sample questions: style (modern/retro/minimal/Other), launch deadline (1w/2-4w/6+w/Other), copy language (EN/HU/bilingual/Other).  
   - Expected JSON (example answers):  
     ```json
     {"user_answers":{"style":"modern","deadline":"2-4w","language":"bilingual","constraints":{"time":"2-4w","budget":"unknown","platform":"web"},"preferences":{"style":"modern","risk":"unknown"}}}
     ```

## Implementation Notes
- Keep the skill instruction-only; no extra resources needed.
- Maintain concise phrasing to minimize context usage.
