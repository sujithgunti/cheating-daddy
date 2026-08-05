# Prioritize Gemini 2.5 Flash Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Reorder fallback models to place `gemini-2.5-flash` first for lower response latencies, and update project guidelines.

**Architecture:** Change `fallbackModels` in `src/utils/gemini.js` to `['gemini-2.5-flash', 'gemini-3.5-flash', 'gemini-3.1-flash-lite']` and update `AGENTS.md`.

**Tech Stack:** JavaScript, Markdown.

---

### Task 1: Re-order fallback cascade in `src/utils/gemini.js`

**Files:**
- Modify: `d:\open_source\cheating-daddy\src\utils\gemini.js`

- [ ] **Step 1: Update fallbackModels array**
  In `sendToGeminiFallback` (around line 397), update the order to:
  ```javascript
  const fallbackModels = ['gemini-2.5-flash', 'gemini-3.5-flash', 'gemini-3.1-flash-lite'];
  ```

- [ ] **Step 2: Verify module compilation**
  Run: `node -e "require('./src/storage.js'); require('./src/utils/gemini.js');"`
  Expected: Command completes with no errors.

---

### Task 2: Update Fallback Guidelines in `AGENTS.md`

**Files:**
- Modify: `d:\open_source\cheating-daddy\AGENTS.md`

- [ ] **Step 1: Update Fallback Text Model Guidelines**
  Update the fallback text model rules (around line 123) to reflect the priority of `gemini-2.5-flash` for latency optimization.

---

### Task 3: Formatting Check

**Files:**
- Modify: `d:\open_source\cheating-daddy\src\utils\gemini.js`
- Modify: `d:\open_source\cheating-daddy\AGENTS.md`

- [ ] **Step 1: Run Prettier formatter**
  Run: `npx prettier --write src/utils/gemini.js AGENTS.md`
  Expected: Formatting completes with no errors.
