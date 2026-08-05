# 2026-06-13 Gemini Fallback Latency Reduction Design

## Goal
Reduce response times during fallback text generation by replacing the slow `gemma-4-31b-it` model with Google's faster, highly intelligent `gemini-3.5-flash` model. 

## Context
When a Groq key is not configured, the application uses Google's hosted API fallback path. Currently, this fallback calls `gemma-4-31b-it`, resulting in high latency:
* **Time to First Token (TTFT)**: ~10.7 seconds
* **Total Latency**: ~28.7 seconds

Both `gemma-4-31b-it` and `gemini-3.5-flash` share the exact same free tier quotas under Google AI Studio (15 RPM and 1,500 RPD). Switching to `gemini-3.5-flash` maintains identical session capacity (up to 15 full 1-hour sessions per day) while reducing TTFT to ~1.0–1.5 seconds and total latency to ~3.0 seconds.

## Proposed Changes

### `src/utils/gemini.js`
1. **Rename and Re-target**:
   * Rename internal function `sendToGemma(transcription)` to `sendToGeminiFallback(transcription)`.
   * Update the `model` parameter from `'gemma-4-31b-it'` to `'gemini-3.5-flash'` inside the `GoogleGenAI` stream request payload.
2. **Session Logging & Diagnostics**:
   * Update internal logging events:
     * `GEMMA_API_START` -> `GEMINI_FALLBACK_START`
     * `GEMMA_FIRST_TOKEN_RECEIVED` -> `GEMINI_FALLBACK_FIRST_TOKEN`
     * `GEMMA_API_COMPLETE` -> `GEMINI_FALLBACK_COMPLETE`
3. **Usage Accounting**:
   * Update character usage counting from `gemma-4-31b-it` to `gemini-3.5-flash` via `incrementCharUsage('gemini', 'gemini-3.5-flash', ...)`.
4. **Invocation Wiring**:
   * Update all calls referencing `sendToGemma` to call `sendToGeminiFallback` (specifically the VAD silence timer trigger and manual input fallback trigger).

### `src/storage.js`
1. **Default Config & Comments**:
   * Update the default limits template comment to reference `gemini-3.5-flash`.
2. **Backward Compatibility**:
   * In `getTodayLimits()`, ensure that both new limits entries and existing daily records defensively initialize/add `'gemini-3.5-flash'` with `{ chars: 0 }` if not present.

## Verification Plan
1. **Compilation Check**: Run `node -e` on modified modules to verify syntax stability.
2. **Log Verification**: Trigger fallback executions (auto/manual mode without a Groq key) and verify that the session log captures the `GEMINI_FALLBACK_START` and `GEMINI_FALLBACK_COMPLETE` events with low latency (TTFT < 2.0s).
