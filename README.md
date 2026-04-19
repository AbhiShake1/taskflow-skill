# taskflow (Claude Code skill)

Author and run multi-agent orchestration harnesses. Parallelize a task across Claude Code, Codex, Cursor, OpenCode, and Pi — each session picks its own agent and model, so mechanical work runs on cheap models and stakes-high work runs on frontier models. This skill teaches Claude Code *how to author* those harnesses in TypeScript.

Install once and you can say things like:

- "Parallelize this scraping job across four OpenCode shards and merge the output."
- "Make a harness that plans with Opus, generates with Sonnet, and lints with Groq."
- "Set up a multi-agent pipeline that reviews and rewrites these 20 files in parallel."

Claude Code will author a real TypeScript harness under `tasks/` using the fluent API from [`@taskflow-corp/cli`](https://www.npmjs.com/package/@taskflow-corp/cli), preview the phase/session tree with `npm run plan`, and only execute after you confirm the shape.

## Prerequisites

- [ ] **Claude Code** ≥ latest — install via `npm install -g @anthropic-ai/claude-code` or the macOS/Windows app.
- [ ] **Node.js ≥ 20** — check with `node --version`. Install via [nodejs.org](https://nodejs.org/) or `brew install node`.
- [ ] **`@taskflow-corp/cli` installed in the target project** — run `npm install @taskflow-corp/cli` in whatever repo you want the harness to live in. The skill always uses this package.
- [ ] **Provider API keys for whichever agents you invoke** — at minimum `ANTHROPIC_API_KEY` (for claude-code sessions). Others are optional per-session: `OPENAI_API_KEY`, `GROQ_API_KEY`, `CEREBRAS_API_KEY`, `GEMINI_API_KEY`.
- [ ] **Per-agent CLIs if you want them as sessions** — `claude-code`, `codex`, `cursor-agent`, `opencode`, `pi` must be on `PATH` for the adapters that shell out. Unused adapters can stay uninstalled.

## Install

```bash
npx skills add AbhiShake1/taskflow-skill
```

Verify installation:

```bash
npx skills list | grep taskflow
```

(Should show `taskflow` with the description.)

## Usage examples

Say any of these in Claude Code — the skill triggers automatically:

- *"Make a harness that scrapes this sitemap with OpenCode shards and merges with Sonnet."*
- *"Parallelize running these tests across four models and pick the highest score."*
- *"Build a multi-agent pipeline: plan with Opus, apply with Sonnet, verify with Gemini."*
- *"Set up a claude-code/codex/cursor side-by-side for this refactor."*

Claude Code will:

1. Author a `tasks/<name>.ts` file using the fluent async-await API (`taskflow(...).run(...)` → `phase(...)` → `session(...)`).
2. Run `npm run plan -- tasks/<name>.ts` to show the phase/session tree visually.
3. Ask you to confirm before executing.
4. On confirm, run `npm run run tasks/<name>.ts` with the Ink TUI (live stream of sessions, tool calls, token usage).

## What's inside the skill

The `SKILL.md` documents:

- Fluent API: `taskflow`, `phase`, `session`, `defineConfig`.
- All 30+ lifecycle hooks (`beforeSession`, `beforeToolCall`, `verifyTaskComplete`, `afterTaskDone`, etc.) and the `HookCtx` surface.
- Todo subsystem: auto-extraction from `- [ ]` markdown, `collectTodos` hook, `forceGeneration` directive, verify-loop that re-engages the agent until todos complete.
- `scope` config (prepended to every session task) and hierarchical `.agents/taskflow/config.ts` discovery.
- Plugin system for opinionated workflows (video/screenshot proofs via UI-TARS, audit logs, custom verification).
- Agent/model picking heuristics for claude-code, codex, cursor, opencode, pi.

## Related

- **SDK on npm**: [`@taskflow-corp/cli`](https://www.npmjs.com/package/@taskflow-corp/cli)
- **Main repo**: [AbhiShake1/taskflow](https://github.com/AbhiShake1/taskflow)

## Risks & limits

- The harness **runs real AI agents** — each session bills against your provider accounts. Always preview with `npm run plan` before executing a new harness.
- Sessions with overlapping `write:` globs will **throw before any execution starts** (intentional safety). Give each concurrent session a distinct write path.
- Auto-todos + verify-loop will **re-engage the agent up to `maxRetries` times** (default 3) if any checkbox is unchecked. Budget accordingly for long tasks.
- `@taskflow-corp/cli`'s adapters assume the respective CLIs are on `PATH` with default flags. Non-standard installations may need `HARNESS_PI_BIN` etc. overrides.

## Troubleshooting

| Problem | Fix |
|---|---|
| `npx skills add` says "No valid skills found" | Confirm YAML frontmatter in SKILL.md has `name` and `description`; they must not contain unescaped special chars. |
| Harness runs but no events reach the TUI | Check `HARNESS_NO_TTY` isn't set; the TUI falls back to JSONL stdout when no TTY is attached. |
| Session fails with "claim conflict" | Two concurrent sessions declared overlapping `write:` globs. Make them disjoint or serialize the sessions. |
| Structured output rejects with "schema mismatch" | The adapter's final assistant message wasn't valid JSON for the zod schema. Inspect `data/runs/<runId>/leaves/<id>/proof.json` for the raw capture. |
| `npm run plan` shows `?` (unknown) nodes | The static AST walker couldn't resolve a dynamic pattern. Rewrite the pattern to literal `Promise.all([...])` over a `.map` array, or accept the unknown node as a runtime-determined leaf. |
| Provider returns 401 / missing key | The env var for that provider isn't set. Check `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, etc. |

## License

MIT — see `LICENSE`.
