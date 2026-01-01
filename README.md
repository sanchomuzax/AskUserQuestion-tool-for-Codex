# AskUserQuestion Skill for Codex

Instruction-only skill that mirrors Claude Code's AskUserQuestion: when a task is ambiguous or needs user choices, it restates the current understanding, asks 1–5 targeted (multi-choice + Other) questions, pauses for answers, emits a normalized `user_answers` JSON, then continues the original task using those answers.

## Contents
- `ask-user-question/SKILL.md`: the skill definition.

## Quick Use
1) Copy `ask-user-question/` into your Codex skills path (e.g., `~/.codex/skills/`).
2) Or package it: `scripts/package_skill.py ask-user-question` (if you have the Codex skill tools repo on PATH).
3) Restart your Codex client if needed.

## Trigger Cues
- Auto: any ambiguous/missing requirement task or preference-driven choice.
- Explicit cues: "ask user questions", "clarify first", "ask back", "double-check preferences", "interview me".

## Example Prompt
"Ask back: I need a mobile app for my runs." → The agent should summarize, ask 1–5 relevant multiple-choice questions with `Other`, pause, output `user_answers` JSON, then proceed using those answers.
