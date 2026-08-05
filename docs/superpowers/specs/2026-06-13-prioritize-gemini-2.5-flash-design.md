# 2026-06-13 Prioritize Gemini 2.5 Flash Design

## Goal
Improve response speed and reliability of the Gemini fallback text generation pathway by prioritizing `gemini-2.5-flash` over `gemini-3.5-flash`.

## Context
During live sessions where Groq is not active, the system falls back to the Google AI Studio hosted text models. The current cascade ordering is `['gemini-3.5-flash', 'gemini-2.5-flash', 'gemini-3.1-flash-lite']`.
* **Gemini 3.5 Flash** has higher average Time to First Token (TTFT) (~5s–9s) due to higher agentic overhead and server-side traffic.
* **Gemini 2.5 Flash** is highly stable and delivers low latency (~1.9s TTFT), which is critical for real-time interview contexts.
To prioritize speed, we will shift `gemini-2.5-flash` to the primary slot of the cascade, and keep `gemini-3.5-flash` as a secondary fallback.

## Proposed Changes

### `src/utils/gemini.js`
Modify the `fallbackModels` array to place `gemini-2.5-flash` first:
```javascript
const fallbackModels = ['gemini-2.5-flash', 'gemini-3.5-flash', 'gemini-3.1-flash-lite'];
```

### `AGENTS.md`
Update the **Fallback Text Model Guidelines** section to document that `gemini-2.5-flash` is preferred for speed, and `gemini-3.5-flash` is secondary.

## Verification Plan
1. **Compilation Check**: Run static checks (`node -e` command) on changed scripts.
2. **Formatting**: Ensure files comply with Prettier configurations.
3. **Manual verification**: Verify that when triggering fallback text generation, the console output/logs show the request successfully routing to `gemini-2.5-flash` first.
