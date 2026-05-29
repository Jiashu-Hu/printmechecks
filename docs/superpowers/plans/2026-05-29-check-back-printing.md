# Check Back Printing Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a printable standard check back, front/back print buttons, combined Ctrl+P printing, and mutually exclusive mobile/custom endorsement controls.

**Architecture:** Keep the feature inside the existing `CheckPrinter.vue` component, because this app currently has one primary print component and the requested behavior is local to that component. Add `check-back.svg` as a static asset, render it inside the same printable sheet as the front, and use one target-aware print function for front-only, back-only, and full-sheet printing.

**Tech Stack:** Vue 3 single-file component, TypeScript, Vite static assets, browser `window.print()`, existing Bootstrap classes.

---

## File Structure

- Create `printmechecks/src/assets/check-back.svg`: project-owned copy of `/Users/teddy/Downloads/back.svg`, with comments removed or written in English only.
- Modify `printmechecks/src/components/CheckPrinter.vue`: render the combined sheet, add endorsement controls, add back-side endorsement overlay, add minimal local types for the check object, and replace the current print function with a target-based function.
- Modify `printmechecks/src/views/HistoryView.vue`: convert the existing plain script setup block to TypeScript if required for `vue-tsc` to run.
- Modify `printmechecks/src/utilities.ts`: fix the existing invalid `float` annotation if it blocks required TypeScript verification.
- Do not commit `.superpowers/`, `AGENTS.md`, or `docs/repowiki/`; they are unrelated existing/untracked workspace files.

## Verification Precondition

- [ ] **Step 1: Check whether dependencies are installed**

Run:

```bash
cd printmechecks
npm run type-check
```

Expected if dependencies are missing:

```text
sh: vue-tsc: command not found
```

If that appears, run:

```bash
npm install
```

Then continue with the tasks. If `npm install` fails because of network sandboxing, rerun it with escalated network approval.

---

### Task 1: Add the Check Back SVG Asset

**Files:**
- Create: `printmechecks/src/assets/check-back.svg`

- [ ] **Step 1: Copy the provided SVG into the app assets**

Run:

```bash
cp /Users/teddy/Downloads/back.svg printmechecks/src/assets/check-back.svg
```

Expected:

```text
printmechecks/src/assets/check-back.svg exists
```

- [ ] **Step 2: Remove non-English SVG comments**

Open `printmechecks/src/assets/check-back.svg` and remove all SVG comment nodes from the copied file.

```xml
<!-- any copied SVG comment -->
```

The SVG may keep all graphical elements and text nodes. Only comments need cleanup so repository comments remain English.

- [ ] **Step 3: Verify the asset is usable by Vite**

Run:

```bash
rg -n "<!--|[\u4e00-\u9fff]" printmechecks/src/assets/check-back.svg
```

Expected:

```text
no output
```

- [ ] **Step 4: Commit the asset**

Run:

```bash
git add printmechecks/src/assets/check-back.svg
git commit -m "Add check back template asset"
```

---

### Task 2: Add Typed Endorsement State and Rendering Logic

**Files:**
- Modify: `printmechecks/src/components/CheckPrinter.vue`
- Modify: `printmechecks/src/views/HistoryView.vue`
- Modify: `printmechecks/src/utilities.ts`

- [ ] **Step 1: Update imports**

Replace the existing Vue import:

```ts
import { ref, reactive, nextTick, watch, onMounted, onUnmounted } from 'vue'
```

with:

```ts
import { computed, ref, reactive, nextTick, watch, onMounted, onUnmounted } from 'vue'
```

Remove this unused import:

```ts
import print from 'print-js';
```

- [ ] **Step 2: Add local check data types**

Add this near the top of the `<script setup lang="ts">` block, after imports:

```ts
type EndorsementMode = 'none' | 'mobile' | 'custom'

type CheckData = {
    accountHolderName: string
    accountHolderAddress: string
    accountHolderCity: string
    accountHolderState: string
    accountHolderZip: string
    checkNumber: string
    date: string
    bankName: string
    amount: string
    payTo: string
    memo: string
    signature: string
    routingNumber: string
    bankAccountNumber: string
    endorsementMode: EndorsementMode
    endorsementText: string
    lineLength?: number
}

type SavedCheckData = Partial<CheckData>
```

- [ ] **Step 3: Add an endorsement mode normalizer**

Add this after the types:

```ts
function getEndorsementMode(value: unknown): EndorsementMode {
    return value === 'mobile' || value === 'custom' ? value : 'none'
}
```

- [ ] **Step 4: Replace `genNewCheck()` with a typed version**

Replace the full existing `genNewCheck()` function with:

```ts
function genNewCheck (): CheckData {
    const checkList = JSON.parse(localStorage.getItem('checkList') || '[]') as SavedCheckData[]
    const recentCheck = checkList[checkList.length - 1]
    return {
        accountHolderName: recentCheck?.accountHolderName || 'John Smith',
        accountHolderAddress: recentCheck?.accountHolderAddress || '123 Cherry Tree Lane',
        accountHolderCity: recentCheck?.accountHolderCity || 'New York',
        accountHolderState: recentCheck?.accountHolderState || 'NY',
        accountHolderZip: recentCheck?.accountHolderZip || '10001',
        checkNumber: recentCheck?.checkNumber ? `${parseInt(recentCheck.checkNumber, 10) + 1}` : '100',
        date: new Date().toLocaleDateString(),
        bankName: recentCheck?.bankName || 'Bank Name, INC',
        amount: '0.00',
        payTo: 'Michael Johnson',
        memo: recentCheck?.memo || 'Rent',
        signature: recentCheck?.signature || 'John Smith',
        routingNumber: recentCheck?.routingNumber || '022303659',
        bankAccountNumber: recentCheck?.bankAccountNumber || '000000000000',
        endorsementMode: getEndorsementMode(recentCheck?.endorsementMode),
        endorsementText: recentCheck?.endorsementText || '',
    }
}
```

- [ ] **Step 5: Add computed back-side endorsement lines**

Replace:

```ts
const line = ref(null)
```

with:

```ts
const line = ref<HTMLElement | null>(null)
```

Then add this after it:

```ts
const backEndorsementLines = computed(() => {
    if (check.endorsementMode === 'mobile') {
        return ['For Mobile Deposit Only', check.signature].filter(Boolean)
    }

    if (check.endorsementMode === 'custom') {
        return [check.endorsementText, check.signature].filter(Boolean)
    }

    return [check.signature].filter(Boolean)
})
```

- [ ] **Step 6: Save a plain check object to history**

Replace:

```ts
checkList.push(check)
```

inside `saveToHistory()` with:

```ts
checkList.push({ ...check })
```

- [ ] **Step 7: Restore endorsement fields from history**

Replace the current `onMounted()` history restore block:

```ts
if (state.check) {
    check.accountHolderName = state.check.accountHolderName
    check.accountHolderAddress = state.check.accountHolderAddress
    check.accountHolderCity = state.check.accountHolderCity
    check.accountHolderState = state.check.accountHolderState
    check.accountHolderZip = state.check.accountHolderZip
    check.checkNumber = state.check.checkNumber
    check.date = state.check.date
    check.bankName = state.check.bankName
    check.amount = state.check.amount
    check.payTo = state.check.payTo
    check.memo = state.check.memo
    check.signature = state.check.signature
    check.routingNumber = state.check.routingNumber
    check.bankAccountNumber = state.check.bankAccountNumber
}
```

with:

```ts
const savedCheck = state.check as SavedCheckData | null
if (savedCheck) {
    check.accountHolderName = savedCheck.accountHolderName || check.accountHolderName
    check.accountHolderAddress = savedCheck.accountHolderAddress || check.accountHolderAddress
    check.accountHolderCity = savedCheck.accountHolderCity || check.accountHolderCity
    check.accountHolderState = savedCheck.accountHolderState || check.accountHolderState
    check.accountHolderZip = savedCheck.accountHolderZip || check.accountHolderZip
    check.checkNumber = savedCheck.checkNumber || check.checkNumber
    check.date = savedCheck.date || check.date
    check.bankName = savedCheck.bankName || check.bankName
    check.amount = savedCheck.amount || check.amount
    check.payTo = savedCheck.payTo || check.payTo
    check.memo = savedCheck.memo || check.memo
    check.signature = savedCheck.signature || check.signature
    check.routingNumber = savedCheck.routingNumber || check.routingNumber
    check.bankAccountNumber = savedCheck.bankAccountNumber || check.bankAccountNumber
    check.endorsementMode = getEndorsementMode(savedCheck.endorsementMode)
    check.endorsementText = savedCheck.endorsementText || ''
}
```

- [ ] **Step 8: Add checkbox state updater**

Add this before `handlePrintShortcut()`:

```ts
function updateEndorsementMode(mode: EndorsementMode, event: Event) {
    const checked = (event.target as HTMLInputElement).checked
    check.endorsementMode = checked ? mode : 'none'
}
```

- [ ] **Step 9: Fix the existing invalid utility type if verification reaches it**

In `printmechecks/src/utilities.ts`, replace:

```ts
var numberFloat: float = parseFloat(number)
```

with:

```ts
const numberFloat = parseFloat(number)
```

- [ ] **Step 10: Convert the existing history view script to TypeScript if `vue-tsc` reports `HistoryView.vue.js`**

In `printmechecks/src/views/HistoryView.vue`, replace:

```vue
<script setup>
```

with:

```vue
<script setup lang="ts">
```

Then add these local types after the imports:

```ts
type HistoryItem = {
  id?: string
  checkNumber: string
  amount: string
  payTo: string
  bankAccountNumber: string
}
```

Replace:

```ts
const history = ref([])
```

with:

```ts
const history = ref<HistoryItem[]>([])
```

Replace:

```ts
const deleteItem = (index) => {
```

with:

```ts
const deleteItem = (index: number) => {
```

Replace:

```ts
const viewItem = (index) => {
```

with:

```ts
const viewItem = (index: number) => {
```

- [ ] **Step 11: Run a type check**

Run:

```bash
cd printmechecks
npm run type-check
```

Expected:

```text
no TypeScript errors introduced by endorsement fields or the utility helper
```

If existing project TypeScript errors appear in untouched files, record them before continuing.

---

### Task 3: Render the Combined Front and Back Preview

**Files:**
- Modify: `printmechecks/src/components/CheckPrinter.vue`

- [ ] **Step 1: Replace the printable preview wrapper**

Replace the top preview structure:

```vue
<div class="wrapper" id="wrapper" style="position: relative;">
    <div class="check-box" id="check-box">
        <div style="position: relative;" id="check-box-print">
```

with:

```vue
<div class="wrapper" id="wrapper">
    <div class="check-sheet" id="check-sheet">
        <div class="check-front" id="check-front">
            <div class="check-front-print" id="check-box-print">
```

- [ ] **Step 2: Close the front region and add the back region**

Immediately after the existing banking block closes, replace the two closing `</div>` tags that end the old preview with this structure:

```vue
            </div>
        </div>
        <div class="check-cut-gap"></div>
        <div class="check-back" id="check-back">
            <img src="@/assets/check-back.svg" class="check-back-template" alt="">
            <div class="back-endorsement-data">
                <div v-for="line in backEndorsementLines" :key="line">{{ line }}</div>
            </div>
        </div>
    </div>
```

The following `<div class="check-data"...>` remains after the new `check-sheet`.

- [ ] **Step 3: Remove absolute positioning from the editor wrapper**

Replace:

```vue
<div class="check-data" style="position: absolute; top: 450px">
```

with:

```vue
<div class="check-data">
```

- [ ] **Step 4: Add the endorsement controls to the form**

In the last form section, after the signature input column, add:

```vue
<div class="col-md-2">
    <div class="form-check" style="margin-top: 32px;">
        <input
            class="form-check-input"
            type="checkbox"
            id="mobileDeposit"
            :checked="check.endorsementMode === 'mobile'"
            :disabled="check.endorsementMode === 'custom'"
            @change="updateEndorsementMode('mobile', $event)"
        >
        <label class="form-check-label" for="mobileDeposit">Mobile</label>
    </div>
</div>
<div class="col-md-2">
    <div class="form-check" style="margin-top: 32px;">
        <input
            class="form-check-input"
            type="checkbox"
            id="customEndorsement"
            :checked="check.endorsementMode === 'custom'"
            :disabled="check.endorsementMode === 'mobile'"
            @change="updateEndorsementMode('custom', $event)"
        >
        <label class="form-check-label" for="customEndorsement">Endorse</label>
    </div>
</div>
<div class="col-md-6" v-if="check.endorsementMode === 'custom'">
    <label for="endorsementText" class="form-label">Endorsement</label>
    <input
        type="text"
        class="form-control"
        id="endorsementText"
        v-model="check.endorsementText"
    >
</div>
```

- [ ] **Step 5: Replace the print button row**

Replace:

```vue
<button type="button" style="float: right;" class="btn btn-primary" @click="printCheck">Print (Ctrl + P)</button>
```

with:

```vue
<div style="float: right;">
    <button type="button" class="btn btn-primary" style="margin-right: 10px;" @click="printCheck('front')">Print Front</button>
    <button type="button" class="btn btn-primary" @click="printCheck('back')">Print Back</button>
</div>
```

- [ ] **Step 6: Verify the template compiles**

Run:

```bash
cd printmechecks
npm run type-check
```

Expected:

```text
the Vue template compiles with the new SVG asset and endorsement controls
```

---

### Task 4: Add Target-Based Print Behavior

**Files:**
- Modify: `printmechecks/src/components/CheckPrinter.vue`

- [ ] **Step 1: Add the print target type**

Add after `type EndorsementMode = 'none' | 'mobile' | 'custom'`:

```ts
type PrintTarget = 'sheet' | 'front' | 'back'
```

- [ ] **Step 2: Replace `printCheck()`**

Replace the current `printCheck()` function with:

```ts
function printCheck (target: PrintTarget = 'sheet') {
    const selectorByTarget = {
        sheet: '#check-sheet',
        front: '#check-front',
        back: '#check-back',
    }
    const targetSelector = selectorByTarget[target]
    const style = document.createElement('style');
    style.textContent = `
      @media print {
        @page {
          margin: 0;
        }
        body {
          margin: 0;
          padding: 0;
          background: white;
        }
        body * {
          visibility: hidden !important;
        }
        ${targetSelector},
        ${targetSelector} * {
          visibility: visible !important;
        }
        ${targetSelector} {
          position: fixed !important;
          top: 0;
          left: 0;
          margin: 0 !important;
          border: none !important;
          box-shadow: none !important;
          background-color: white !important;
        }
        .check-front,
        .check-back {
          break-inside: avoid;
          page-break-inside: avoid;
        }
        ${target === 'sheet' ? '.check-cut-gap { visibility: visible !important; }' : ''}
      }
    `;
    document.head.appendChild(style);
    window.print();
    style.remove();
}
```

- [ ] **Step 3: Update Ctrl+P behavior**

Replace:

```ts
printCheck();
```

inside `handlePrintShortcut()` with:

```ts
printCheck('sheet');
```

- [ ] **Step 4: Run a type check**

Run:

```bash
cd printmechecks
npm run type-check
```

Expected:

```text
no TypeScript errors from the print target changes
```

---

### Task 5: Update Styles for Sheet, Front, Back, and Endorsement Text

**Files:**
- Modify: `printmechecks/src/components/CheckPrinter.vue`

- [ ] **Step 1: Replace the old `.check-box` styles**

Replace:

```css
.check-box {
    width: 1200px;
    height: 1553px;
    border: 1px solid #e6e6e6;
    background-color: white;
    margin: 0 auto;
    background: url('../assets/checkbg.png');
    background-repeat: no-repeat;
    background-size: contain;
}

#check-box {
    width: 100%;
}
```

with:

```css
.check-sheet {
    width: 1200px;
    margin: 0 auto;
    background-color: white;
}

.check-front,
.check-back {
    position: relative;
    width: 1200px;
    height: 462px;
    border: 1px solid #e6e6e6;
    background-color: white;
    box-sizing: border-box;
}

.check-front {
    background: url('../assets/checkbg.png');
    background-repeat: no-repeat;
    background-size: contain;
}

.check-front-print {
    position: relative;
}

.check-cut-gap {
    height: 50px;
    border-left: 1px dashed #c7c7c7;
    border-right: 1px dashed #c7c7c7;
    background-color: white;
}

.check-back {
    overflow: hidden;
}

.check-back-template {
    position: absolute;
    top: 50%;
    left: 50%;
    width: 462px;
    height: 1048px;
    transform: translate(-50%, -50%) rotate(90deg);
    transform-origin: center center;
}

.back-endorsement-data {
    position: absolute;
    top: 72px;
    right: 112px;
    width: 310px;
    font-family: Caveat;
    font-size: 34px;
    line-height: 1.15;
    transform: rotate(-2deg);
    white-space: pre-line;
}
```

- [ ] **Step 2: Build the app**

Run:

```bash
cd printmechecks
npm run build
```

Expected:

```text
vite build completes and includes check-back.svg in the built assets
```

---

### Task 6: Manual Browser Verification

**Files:**
- No source edits unless verification finds a defect.

- [ ] **Step 1: Start the dev server**

Run:

```bash
cd printmechecks
npm run dev
```

Expected:

```text
VITE ready and a localhost URL, usually http://localhost:5173/
```

- [ ] **Step 2: Verify the default preview**

Open the Vite URL and check:

```text
front check appears at the top
blank cutting gap appears below the front
standard check back appears below the gap
endorsement text appears over the back endorsement area
```

- [ ] **Step 3: Verify endorsement modes**

Use the form and check:

```text
no checkbox selected: back shows signature only
Mobile selected: Endorse is disabled, back shows "For Mobile Deposit Only" and signature
Mobile unchecked: Endorse becomes enabled
Endorse selected: Mobile is disabled and custom endorsement input appears
custom endorsement entered: back shows custom text and signature
Endorse unchecked: Mobile becomes enabled and back shows signature only
```

- [ ] **Step 4: Verify print targets**

Use browser print preview for:

```text
Print Front: preview contains only the front
Print Back: preview contains only the back
Ctrl+P: preview contains the combined sheet with front, gap, and back
```

- [ ] **Step 5: Verify history restore**

Save a check with `Mobile`, reopen it from History, and confirm `Mobile` is selected and the back text is restored. Repeat with `Endorse` and a custom endorsement value.

---

### Task 7: Final Checks and Commit

**Files:**
- Commit source and asset files changed during implementation.

- [ ] **Step 1: Run required commands**

Run:

```bash
cd printmechecks
npm run type-check
npm run build
npm run lint
```

Expected:

```text
all three commands complete without errors
```

- [ ] **Step 2: Review the diff**

Run:

```bash
git diff -- printmechecks/src/components/CheckPrinter.vue printmechecks/src/assets/check-back.svg printmechecks/src/views/HistoryView.vue printmechecks/src/utilities.ts
git status --short
```

Expected:

```text
the diff is limited to the check back printing feature
.superpowers/, AGENTS.md, and docs/repowiki/ remain untracked and unstaged
```

- [ ] **Step 3: Commit the implementation**

Run:

```bash
git add printmechecks/src/components/CheckPrinter.vue printmechecks/src/assets/check-back.svg printmechecks/src/views/HistoryView.vue printmechecks/src/utilities.ts
git commit -m "Add check back printing"
```

Expected:

```text
implementation commit is created after verification
```
