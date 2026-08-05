# Gemini Fallback Latency Reduction Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Reduce response latency during fallback text generation by replacing the slow `gemma-4-31b-it` model with the faster `gemini-3.5-flash` model.

**Architecture:** We will modify the fallback pathway to send prompts to `gemini-3.5-flash` instead of `gemma-4-31b-it`. We will adjust session logging and storage metric counters to align with this model name, ensuring smooth backward compatibility for users with existing metadata.

**Tech Stack:** JavaScript, Electron, Google GenAI SDK.

---

### Task 1: Update Storage limits logic in `src/storage.js`

**Files:**
- Modify: `d:\open_source\cheating-daddy\src\storage.js`

- [ ] **Step 1: Update limits default template comment**
  Update the comment on `DEFAULT_LIMITS` (line 39) to reference `gemini-3.5-flash`:
  ```javascript
  const DEFAULT_LIMITS = {
      data: [] // Array of { date: 'YYYY-MM-DD', flash: { count }, flashLite: { count }, groq: { 'qwen3-32b': { chars, limit }, 'gpt-oss-120b': { chars, limit }, 'gpt-oss-20b': { chars, limit } }, gemini: { 'gemini-3.5-flash': { chars } } }
  };
  ```

- [ ] **Step 2: Update today limit initialization and compatibility in `getTodayLimits`**
  Modify lines 290–294 to check for `'gemini-3.5-flash'` defensively and initialize it:
  ```javascript
          if(!todayEntry.gemini) {
              todayEntry.gemini = {
                  'gemini-3.5-flash': { chars: 0 }
              };
          } else if (!todayEntry.gemini['gemini-3.5-flash']) {
              todayEntry.gemini['gemini-3.5-flash'] = { chars: 0 };
          }
  ```

- [ ] **Step 3: Update `newEntry` configuration**
  Update the fallback entry template in `getTodayLimits` (lines 311–313):
  ```javascript
          gemini: {
              'gemini-3.5-flash': { chars: 0 }
          }
  ```

- [ ] **Step 4: Verify storage module compilation**
  Run: `node -e "require('./src/storage.js')"`
  Expected: Command finishes with no errors.

---

### Task 2: Replace model execution logic in `src/utils/gemini.js`

**Files:**
- Modify: `d:\open_source\cheating-daddy\src\utils\gemini.js`

- [ ] **Step 1: Rename function and re-target model**
  Rename function `sendToGemma(transcription)` to `sendToGeminiFallback(transcription)` and update the `model` in the `generateContentStream` payload to `'gemini-3.5-flash'`:
  ```javascript
  async function sendToGeminiFallback(transcription) {
      // ...
      appendSessionLog(currentSessionId, 'GEMINI_FALLBACK_START', { model: 'gemini-3.5-flash' });
      // ...
      const response = await ai.models.generateContentStream({
          model: 'gemini-3.5-flash',
          contents: messagesWithSystem,
      });
  ```

- [ ] **Step 2: Update logging and usage metrics**
  Within the renamed function, update all `GEMMA_` logs to `GEMINI_FALLBACK_` logs and update `incrementCharUsage`:
  ```javascript
                  if (isFirst) {
                      appendSessionLog(currentSessionId, 'GEMINI_FALLBACK_FIRST_TOKEN', { tokenChars: chunkText.length });
                  }
      // ...
      incrementCharUsage('gemini', 'gemini-3.5-flash', inputChars + outputChars);
      // ...
      appendSessionLog(currentSessionId, 'GEMINI_FALLBACK_COMPLETE', { chars: fullText.length });
  ```

- [ ] **Step 3: Update invocation references**
  Update calling sites to target the new function name `sendToGeminiFallback`:
  * Inside `initializeGeminiSession` (VAD silence timer trigger):
    ```javascript
                                      if (hasGroqKey()) {
                                          sendToGroq(currentTranscription);
                                      } else {
                                          sendToGeminiFallback(currentTranscription);
                                      }
    ```
  * Inside `ipcMain.handle('send-text-message')` or other manual trigger paths (around line 1105):
    ```javascript
                  sendToGeminiFallback(text.trim());
    ```

- [ ] **Step 4: Verify module compilation**
  Run: `node -e "require('./src/storage.js'); require('./src/utils/gemini.js');"`
  Expected: Command finishes with no errors.

---

### Task 3: Formatting Check

**Files:**
- Modify: `d:\open_source\cheating-daddy\src\storage.js`, `d:\open_source\cheating-daddy\src\utils\gemini.js`

- [ ] **Step 1: Run Prettier formatter**
  Run: `npx prettier --write src/storage.js src/utils/gemini.js`
  Expected: Formatting succeeds without errors.
