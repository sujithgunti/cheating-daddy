# Live Session Status Updates Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Provide detailed live status updates (`Speech detected...`, `Thinking...`, and `Responding...`) in the transparent window status bar.

**Architecture:** Inject `sendToRenderer('update-status', ...)` calls at key lifecycle junctions (VAD speech arrival, API start, and first-token arrival) in the main process's Gemini/Groq execution paths.

**Tech Stack:** JavaScript, Electron IPC.

---

### Task 1: Update Status logic in `src/utils/gemini.js`

**Files:**
- Modify: `d:\open_source\cheating-daddy\src\utils\gemini.js`

- [ ] **Step 1: Inject VAD Speech Detected status**
  Inside `initializeGeminiSession` (around line 520), update the input transcription listener block to emit `'Speech detected...'`:
  ```javascript
                      if (newText.trim() !== '') {
                          currentTranscription += newText;
                          sendToRenderer('transcription-update', currentTranscription);
                          sendToRenderer('update-status', 'Speech detected...');
  ```

- [ ] **Step 2: Inject "Thinking..." status into Groq execution**
  At the beginning of `sendToGroq` (around line 260), add status update:
  ```javascript
      console.log(`Sending to Groq (${modelToUse}):`, transcription.substring(0, 100) + '...');
      sendToRenderer('update-status', 'Thinking...');
  ```

- [ ] **Step 3: Inject "Thinking..." status into Gemini Fallback execution**
  At the beginning of `sendToGeminiFallback` (around line 380), add status update:
  ```javascript
      console.log('Sending to Gemini fallback:', transcription.substring(0, 100) + '...');
      sendToRenderer('update-status', 'Thinking...');
  ```

- [ ] **Step 4: Inject "Responding..." status into Groq stream handler**
  Inside the JSON streaming chunk loop of `sendToGroq` (around line 320), add the status update on first token:
  ```javascript
                          const token = json.choices?.[0]?.delta?.content || '';
                          if (token) {
                              if (isFirst) {
                                  appendSessionLog(currentSessionId, 'GROQ_FIRST_TOKEN_RECEIVED', { tokenChars: token.length });
                                  sendToRenderer('update-status', 'Responding...');
                              }
  ```

- [ ] **Step 5: Inject "Responding..." status into Gemini Fallback stream handler**
  Inside the generator chunk loop of `sendToGeminiFallback` (around line 430), add the status update on first token:
  ```javascript
              for await (const chunk of response) {
                  const chunkText = chunk.text;
                  if (chunkText) {
                      if (isFirst) {
                          appendSessionLog(currentSessionId, 'GEMINI_FALLBACK_FIRST_TOKEN', { tokenChars: chunkText.length });
                          sendToRenderer('update-status', 'Responding...');
                      }
  ```

- [ ] **Step 6: Verify module compilation**
  Run: `node -e "require('./src/storage.js'); require('./src/utils/gemini.js');"`
  Expected: Command completes with no errors.

---

### Task 2: Formatting Check

**Files:**
- Modify: `d:\open_source\cheating-daddy\src\utils\gemini.js`

- [ ] **Step 1: Run Prettier formatter**
  Run: `npx prettier --write src/utils/gemini.js`
  Expected: Formatting completes with no errors.
