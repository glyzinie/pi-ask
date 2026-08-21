# pi-ask

A small interactive `ask` tool for pi-coding-agent.

## Why

It intentionally avoids copying the large `questionnaire.ts` custom TUI.
Instead it composes Pi's built-in `ctx.ui.select()` and `ctx.ui.input()` dialogs.

Model-facing parameters are deliberately small:

```ts
ask({
  questions: [
    {
      prompt: "Which database should we use?",
      options: ["PostgreSQL", "SQLite"]
    }
  ]
})
```

- `options: []` opens free-text input directly.
- A non-empty option list automatically gets an `Other…` choice.
- Up to 4 questions are accepted per call.
- Up to 8 options are accepted per question.
- The tool is sequential so an interactive question is not run concurrently with another tool batch.

## Context compaction

After an `ask` call has a result, the `context` hook rewrites only the copy sent to the LLM.
The session JSONL remains unchanged.

Instead of repeatedly sending this on later turns:

```text
assistant tool call: ask({ prompt + all options ... })
tool result: Q1: SQLite
```

future model calls see approximately:

```text
[User decisions]
- Which database should we use? → SQLite
```

This removes rejected options from future context while preserving the user's actual decision.

## Install

Copy `pi-ask.ts` to:

```text
~/.pi/agent/extensions/pi-ask.ts
```

Then reload/restart Pi.

The extension imports only:

- `@earendil-works/pi-coding-agent` (type only)
- `@earendil-works/pi-agent-core` (type only)
- `typebox`

It does not directly depend on `@earendil-works/pi-tui`.

## Suggested agent instruction

You usually do not need a long prompt guideline. If your model rarely invokes `ask`, a short instruction in your existing agent instructions is enough, for example:

```text
Use the ask tool when a user decision materially affects the implementation.
```
