# 2026-06-13 Live Session Status Updates Design

## Goal
Improve live session visual feedback by updating the transparent window status bar to show distinct pipeline states (`Speech detected...`, `Thinking...`, and `Responding...`) instead of only showing `Listening...`.

## Context
During a live session, the status bar in the top-right corner stays on `Listening...` even while the interviewer is speaking, the API is querying, or the response is actively streaming. Reusing the existing `update-status` IPC event to broadcast detailed states at key transition points in `src/utils/gemini.js` will resolve this lack of visual feedback.

## Proposed Changes

### `src/utils/gemini.js`
1. **Speech Detection**:
   * Inside the `onmessage` callback within `initializeGeminiSession()`, send `update-status` with `'Speech detected...'` when input transcription is received.
2. **Thinking / Querying**:
   * At the beginning of `sendToGroq()` and `sendToGeminiFallback()`, send `update-status` with `'Thinking...'`.
3. **Responding / Streaming**:
   * Inside the streaming loops of both `sendToGroq()` and `sendToGeminiFallback()`, send `update-status` with `'Responding...'` when the first chunk of the response is received (`isFirst === true`).

## Verification Plan
1. **Compilation Check**: Run `node -e` on modified modules to verify syntax stability.
2. **Runtime Verification**: Run `npm start`, enter a session, trigger transcription inputs, and observe the status bar in the top right corner transitions dynamically:
   * Speak ➔ Shows `Speech detected...`
   * Pause/Trigger ➔ Shows `Thinking...`
   * Stream starts ➔ Shows `Responding...`
   * Complete ➔ Returns to `Listening...`
