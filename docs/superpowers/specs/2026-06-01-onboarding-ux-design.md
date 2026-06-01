# Onboarding UX Redesign

**Date:** 2026-06-01
**Focus:** First-run / onboarding experience
**Approach:** Docs-first path splitter with real-time CLI savings feedback

---

## Problem

The current onboarding has a delayed value problem. The "aha moment" — seeing real token savings — arrives at step 4 of the quickstart, after the user has already written code, made a library call, and sent a request to an LLM. For the Agent User persona, the proxy and wrap modes are buried below the library-first content, meaning many power users never find the fastest path to value.

The sidebar is a flat list of 35+ pages with no structure that signals "start here" or "this section is for you."

---

## Personas

Two distinct personas share the onboarding entry point and must be branched early.

**Agent User** — uses Claude Code, Cursor, or Codex daily. Not building an app. Wants cheaper, faster sessions with zero friction. Their path: `headroom wrap <agent>`. Their blockers: trust ("will it interfere?") and invisible value ("is it doing anything?").

**App Builder** — building an LLM-powered product. Arrives via PyPI or npm. Wants to drop `compress()` into their existing code. Their blockers: picking the right integration pattern and understanding the TypeScript proxy requirement.

---

## Design

### Section 1: Quickstart page restructure

The quickstart page (`docs/content/docs/quickstart.mdx`) is restructured around a role selector that appears **above the fold** — the first element on the page.

Role selector (implemented using the existing fumadocs `<Tabs>` component, same pattern as the Python/TypeScript language tabs):

```text
I want to…
[ Save tokens on my Claude Code / Cursor / Codex sessions ]   [ Add compression to my app ]
```

Selecting a track renders only that track's content. Both tracks are 3 steps, with the aha moment at step 3.

#### Track A — Agent User (target: ~60 seconds)

1. `pip install "headroom-ai[proxy]"`
2. `headroom wrap claude` (auto-starts proxy, wires Claude Code)
3. Inline savings appear in terminal ← aha moment

Next steps: `headroom learn` · persistent install · cross-agent memory

#### Track B — App Builder (target: ~90 seconds)

1. `pip install headroom-ai` / `npm install headroom-ai`
2. `compress(messages, model="…")` — minimal single-function example
3. `print(result.tokens_saved)` ← aha moment, already in the result object

Next steps: proxy mode · SDK wrappers · LangChain · Vercel AI SDK

#### Key changes from today

- Role selector is above the fold, not a buried "Alternative" section
- Aha moment moves from step 4 → step 3 in both tracks
- Power users never see library code; app builders never see wrap
- Each track ends with focused next-step cards pointing into that persona's journey
- Proxy mode is step 2 of Track A, not an afterthought at the bottom

### Section 2: CLI savings output

The Agent User aha moment lives in the terminal. Two new output blocks are added to `headroom wrap` (and the proxy startup path).

**First-compression banner** — fires once per session, on the first compression only:

```text
━━━ Headroom ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
▶ First compression: 17,765 → 1,408 tokens  92% saved
  Transforms: SmartCrusher · CacheAligner
  ≈ $0.048 saved · session total so far: $0.048
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Session-end summary** — always shown when the wrapped session exits:

```text
━━━ Headroom session summary ━━━━━━━━━━━━━━━━━━━━
 Compressions: 24  ·  Tokens saved: 141,230  ·  Cost saved: ≈ $0.42
 Run `headroom learn` to extract lessons from this session.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Design rules:

- Banner fires **once per session** on first compression — not every call (avoids polluting agent output)
- Subsequent compressions are silent
- Session-end summary always appears when the session exits
- Cost estimate shown by default; suppressible via config
- `headroom learn` nudge at session end surfaces a high-value hidden feature naturally

**App Builder equivalent:** No CLI change needed. The `compress()` return object already carries `.tokens_saved`, `.tokens_before`, `.tokens_after`, `.compression_ratio`. The quickstart step 3 is just a `print()` call — the data is already there.

### Section 3: Docs information architecture

The flat 35-page sidebar (controlled by `docs/content/docs/meta.json`) is regrouped into 5 sections. No pages are deleted — only reorganized. Existing URLs are preserved (or redirected if moved).

Proposed sidebar structure:

```text
Get Started
  → Quickstart   ← role selector lives here
    Installation

Agent Users
    Wrap an agent
    Persistent install
    Failure learning (headroom learn)
    Cross-agent memory

App Builders
    Python library
    TypeScript SDK
    Proxy server
    Framework integrations
      LangChain · Agno · Strands · Vercel AI SDK · Anthropic SDK · OpenAI SDK · LiteLLM

How It Works
    Architecture · CCR · SmartCrusher · CodeCompressor · Cache Optimization
    Context Management · Image Compression · Simulation

Reference
    Configuration · API Reference · Benchmarks · Metrics · Troubleshooting · Errors
```

**`headroom learn` promotion:** Currently the Failure Learning page exists but is not linked from the quickstart or wrap docs — users only find it by scanning the sidebar. Three changes promote it:

1. Session-end banner nudge: `Run headroom learn to extract lessons from this session.`
2. Agent User track's next-step cards include a direct link
3. Sidebar places it prominently under Agent Users

---

## Out of scope

- Browser playground / live demo — deferred; high implementation cost
- Dashboard (headroomlabs.ai) changes — separate surface, separate effort
- CLI flags or compression algorithm changes
- Any changes to the TypeScript proxy requirement (addressed in docs framing only)

---

## Success criteria

- A new Agent User reaches their first visible token saving in ≤ 3 steps / ≤ 60 seconds
- A new App Builder reaches their first `tokens_saved` output in ≤ 3 steps / ≤ 90 seconds
- `headroom learn` discovery rate increases (measurable via telemetry on `headroom learn` invocations)
- No existing docs content is lost or broken

---

## Files to change

| File | Change |
| ---- | ------ |
| `docs/content/docs/quickstart.mdx` | Full restructure — role selector + two tracks |
| `docs/content/docs/meta.json` | Regroup into 5 sidebar sections |
| `headroom/cli/proxy.py` | Add first-compression banner output |
| `headroom/cli/` (wrap shared scaffolding) | Add session-end summary block |
| `docs/content/docs/persistent-installs.mdx` | Move under Agent Users nav group |
| `docs/content/docs/failure-learning.mdx` | Move under Agent Users nav group |
