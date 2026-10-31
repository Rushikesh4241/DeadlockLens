# DeadlockLens - Interactive Banker's Algorithm & Deadlock Analysis Dashboard

> **University Operating Systems Activity 2:** *"AI-Assisted Dashboard Development and Deadlock Analysis"*  
> **Course:** CS304 - Operating Systems  
> **Evaluation Criteria:** Algorithmic correctness, complete coverage of 6 mandatory test cases (TC1–TC6), pedagogical clarity of mathematical explanations, mission-control UI design, and live-demo readiness.

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Key Features by Module](#key-features-by-module)
3. [Pure Algorithmic Specifications](#pure-algorithmic-specifications)
4. [Mandatory Test Cases Verification (TC1–TC6)](#mandatory-test-cases-verification-tc1tc6)
5. [How to Run Locally](#how-to-run-locally)
6. [How to Deploy to GitHub Pages in 4 Steps](#how-to-deploy-to-github-pages-in-4-steps)
7. [System Architecture & Design Standards](#system-architecture--design-standards)

---

## Project Overview

**DeadlockLens** is a single self-contained, zero-dependency interactive web dashboard engineered for computer science students and educators exploring concurrency, resource allocation, and deadlock phenomena.

- **Offline-First & Self-Contained:** Built entirely in clean, semantic HTML5, modern CSS3, and bare-metal vanilla JavaScript (ES6+). Zero external CDNs, zero third-party font/icon files, zero network fetches, and no build or bundling step. Double-click `index.html` to run in any modern web browser.
- **Strict Decoupling:** Algorithmic logic (`DeadlockAlgorithms`) is completely separated from DOM manipulation (`UIController`), enabling straightforward mathematical verification and automated unit testing.
- **Mission-Control Visual Design:** Glassmorphic translucent cards (`backdrop-filter: blur(14px)`), deep navy theme (`#0A0F1E`) with high-contrast light theme toggle (`#F8FAFC`), distinct process color identities ($P_0$–$P_7$), and smooth easing transitions respecting `prefers-reduced-motion`.
- **Full Keyboard Accessibility:** Instant tab switching using numeric keys `1`–`7`, spacebar to play/pause stepping, arrow keys for forward/back stepping, and `T` for instant theme toggling.
- **State Persistence:** Automatic state synchronization to `localStorage` wrapped in defensive `try/catch` handlers with JSON export and import capabilities.

---

## Key Features by Module

### 1. System Setup (Tab 1 • Key `1`)
- **Configurable Capacity:** Supports $1$ to $8$ processes ($P_0$ through $P_7$) and $1$ to $6$ resource types ($A$ through $F$). Default configuration: $5$ processes $\times$ $3$ resource types.
- **Dynamic Matrix Grid:**
  - **Allocation Matrix:** Quantities currently allocated to each process.
  - **Maximum Matrix:** Declared upper-bound claim of each process.
  - **Live Need Matrix (Read-Only):** Automatically computed in real time ($\text{Need} = \text{Maximum} - \text{Allocation}$). Highlights zero-need processes ready to complete immediately.
  - **Available Vector:** Free unallocated resource instances ready for assignment.
  - **Total System Vector (Read-Only):** $\text{Total}[r] = \text{Available}[r] + \sum_i \text{Allocation}[i][r]$.
- **Scenario Presets:** One-click loading for "TC1 - Safe State", "Deadlock Scenario", "No-deadlock baseline", and custom configurations.
- **Live Input Validation:** Real-time cell scanning. Any violation (negative number, non-integer, blank cell, or $\text{Allocation} > \text{Maximum}$) triggers an inline warning banner and causes the invalid cell to pulse with a red glow, blocking analysis until resolved.

### 2. Safety Analysis (Tab 2 • Key `2`)
- **Banker's Safety Stepper:** Interactive visual execution player with Play, Pause, Step Forward, Step Backward, Speed adjustment ($0.5\times$ to $3.0\times$), and Jump to End.
- **Strict Multi-Pass Scan Rule:** Scans processes $P_0 \to P_{n-1}$ in sequential index order across full passes without resetting to $P_0$ mid-pass, repeating passes until no further progress occurs.
- **Per-Step Execution Spotlight:**
  - Active candidate process chip in its signature color.
  - Resource-by-resource verification pills with green $\checkmark$ or red $\times$ badges ($\text{Need}[i][r] \le \text{Work}[r]$).
  - Number tweening comparing $\text{Work Before} \to +\text{Release} \to \text{Work After}$.
  - Real-time animated fly-in of safe sequence chips ($\langle P_1 \rangle \to \langle P_3 \rangle \to \dots$).
  - Plain-English explanation banner quoting exact numbers (e.g., *"P2 cannot finish: Need A=6 > Work A=5"*).
- **Outcome Banners:** Green celebration banner with drawn SVG checkmark for `SAFE`, or alert banner with gentle shake animation for `UNSAFE` detailing stuck processes.

### 3. Resource Request Evaluation (Tab 3 • Key `3`)
- **Dynamic Request Composer:** Process selector with numeric steppers for each resource type and quick-fill presets for TC2 (Wait) and TC3 (Unsafe / Safe).
- **Strict 5-Step Banker's Validation:**
  1. Process ID existence check ($P_0 \le P_i \le P_{n-1}$).
  2. Non-negative whole number verification.
  3. Maximum claim constraint: $\text{Request}[i] \le \text{Need}[i]$ (prevents claim overdraft).
  4. Physical availability check: $\text{Request}[i] \le \text{Available}$ (determines if process must wait).
  5. Tentative provisional allocation ($\text{Avail} - \text{Req}$, $\text{Alloc} + \text{Req}$, $\text{Need} - \text{Req}$) and safety algorithm simulation.
- **Side-by-Side Diff Matrices:** "Before", "Tentative", and "Final Action" matrices side-by-side with animated pulsing cyan borders on modified cells.
- **Rollback vs Commit:** Safely rolls back state on unsafe requests with message `"Request DENIED: granting it would lead to an unsafe state. State rolled back."`, and permits 1-click permanent commit on `GRANTED`.

### 4. Deadlock Detection & Wait-For Graph (Tab 4 • Key `4`)
- **Multi-Instance Matrix Detection:**
  - $\text{Work} = \text{Available}$; $\text{Finish}[i] = (\text{Allocation}[i] == \mathbf{0})$.
  - Successive passes checking $\text{Request}[i] \le \text{Work}$.
  - Detects deadlocked set $\{P_1, P_2, P_3, P_4\}$ and final work vector $(0, 1, 0)$.
- **Procedural SVG Wait-For Graph (WFG):**
  - Circular layout positioning nodes with distinct colors and process labels.
  - Directed curved cubic/quadratic bezier edges with arrowheads. An edge $P_i \to P_j$ is drawn whenever $P_i$ requests a resource held by $P_j$ that exceeds final work.
  - First-cycle isolation via DFS (starting at lowest-index deadlocked process, traversing neighbours in ascending order) highlighted with a pulsing red glow: $P_1 \to P_2 \to P_3 \to P_1$.
- **Dual Recovery Engine:**
  - **Strategy A (Victim Termination):** Identifies victim deadlocked process holding the fewest total resources (tie-breaker: lowest process index). Terminating $P_1$ releases $(2,0,0)$ into Available, enabling $P_0, P_3, P_2, P_4$ to finish without deadlock.
  - **Strategy B (Manual Resource Preemption):** Choose a victim process, select resource quantities to confiscate, and re-run detection to test if deadlock resolves.

### 5. Prevention Lab (Tab 5 • Key `5`)
- **Coffman Condition Proofs:** Interactive cards for Mutual Exclusion, Hold & Wait, No Preemption, and Circular Wait with animated SVG mini-diagrams and toggles.
- **Hold & Wait Elimination (All-or-Nothing):**
  - Blocked processes release all held resources ($(7,1,6)$ released).
  - $P_0$ finishes $\to$ Available reaches $(7,2,6)$.
  - Each blocked process in index order atomically requests its combined full claim ($\text{Allocation} + \text{Request}$): $P_1(4,0,2)$, $P_2(3,0,4)$, $P_3(3,1,1)$, $P_4(0,0,4)$.
  - All granted atomically $\to$ completion sequence $\langle P_0, P_1, P_2, P_3, P_4 \rangle$.
  - Includes detailed trade-off analysis (lower utilization, starvation risk, rollback costs).
- **Circular Wait Elimination (Global Resource Ordering):**
  - Linear order $A (0) < B (1) < C (2) < \dots$.
  - Rule: A process holding resource type $k$ may only request types $\ge k$.
  - Validates all requests on Deadlock Scenario: flags $P_3$ (holds $C$, requests $A$) as an ordering violation; all other processes comply.

### 6. Test Suite (Tab 6 • Key `6`)
- **Automated Verification Runner:** One-click "Run All Tests" executing bare-metal JS algorithms against test cases TC1 through TC6.
- **Interactive Score Ring:** Animated circular SVG progress meter displaying 6/6 (100% PASS).
- **Individual Test Loading:** "Load Test" button on each row instantly sets up the corresponding dashboard scenario.
- **Academic Export Utilities:**
  - Copy results table as GitHub-Flavored Markdown to clipboard.
  - Download complete test execution report as structured JSON.
  - Print-friendly layout (`@media print`) optimized for university submission screenshots.

### 7. Guide & About (Tab 7 • Key `7`)
- **Algorithmic Documentation:** Formulas, pseudocode, and mathematical definitions.
- **Operating Systems Glossary:** Safe State, Unsafe State, Deadlock, Starvation, WFG, Preemption.
- **Editable University Submission Card:** Course, Activity, Student Names, Section, and Date saved in `localStorage`.
- **AI-Assisted Build Disclosure:** University compliance statement.
- **Guided 3-Minute Demo Tour (Demo Mode):** Interactive 8-step modal tour navigating through all dashboard tabs.

---

## Pure Algorithmic Specifications

### 1. Banker's Safety Algorithm
```
Input: Allocation[n][m], Need[n][m], Available[m]
Output: isSafe, safeSequence, steps, workFinal

1. Work = copy(Available)
2. Finish[i] = false for all i in 0..n-1
3. safeSequence = []
4. Repeat:
     progressMade = false
     For i = 0 to n-1:
       If Finish[i] == false:
         If Need[i][r] <= Work[r] for all r in 0..m-1:
           Work[r] = Work[r] + Allocation[i][r] for all r
           Finish[i] = true
           safeSequence.append(i)
           progressMade = true
   Until progressMade == false

5. Return (length(safeSequence) == n), safeSequence
```

### 2. Multi-Instance Deadlock Detection Algorithm
```
Input: Allocation[n][m], Request[n][m], Available[m]
Output: isDeadlocked, deadlockedSet, workFinal

1. Work = copy(Available)
2. For i = 0 to n-1:
     Finish[i] = (Allocation[i] == [0, ..., 0])
3. Repeat:
     progressMade = false
     For i = 0 to n-1:
       If Finish[i] == false:
         If Request[i][r] <= Work[r] for all r in 0..m-1:
           Work[r] = Work[r] + Allocation[i][r] for all r
           Finish[i] = true
           progressMade = true
   Until progressMade == false

4. deadlockedSet = { i | Finish[i] == false }
5. Return (deadlockedSet is not empty), deadlockedSet
```

### 3. Wait-For Graph (WFG) Edge & Cycle Construction
```
1. For each pair Pi, Pj in deadlockedSet (i != j):
     Draw directed edge Pi -> Pj if:
       exists resource r such that Request[i][r] > Work_final[r] and Allocation[j][r] > 0
2. Order adjacency lists in ascending process index order
3. Run Depth-First Search (DFS) starting from lowest index deadlocked process
4. Isolate first encountered cycle: P1 -> P2 -> P3 -> P1
```

---

## Mandatory Test Cases Verification (TC1–TC6)

All 6 acceptance tests run dynamically in the browser against the real algorithms:

| Test ID | Scenario | Expected Outcome | Actual Result (Computed Live) | Status |
| :--- | :--- | :--- | :--- | :---: |
| **TC1** | Safe State Verification (TC1 Preset: Alloc, Max, Avail $(3,3,2)$) | $\text{SAFE}$, sequence $\langle P_1, P_3, P_4, P_0, P_2 \rangle$; Work: $(3,3,2) \to (5,3,2) \to (7,4,3) \to (7,4,5) \to (7,5,5) \to (10,5,7)$ | `SAFE: true`, sequence $\langle P_1, P_3, P_4, P_0, P_2 \rangle$; Work final: $(10,5,7)$ | **PASS** |
| **TC2** | Request Exceeds Available (TC1, $P_2$ requests $(6,0,0)$) | $P_2$ must WAIT ($A: 6 > 3$); State unchanged | `P2 must WAIT: requested A=6 exceeds Available A=3. State unchanged.` | **PASS** |
| **TC3** | Request Makes State Unsafe & Companion Safe Grant ($P_4$ requests $(3,3,0)$; $P_1$ requests $(1,0,2)$) | $P_4$: Request DENIED (unsafe) and rolled back; $P_1$: GRANTED with safe sequence $\langle P_1, P_3, P_4, P_0, P_2 \rangle$ | `P4: Request DENIED: granting it would lead to an unsafe state. State rolled back.; P1: Request GRANTED: state remains safe with safe sequence <P1, P3, P4, P0, P2>.` | **PASS** |
| **TC4** | Multi-Instance Deadlock Detection & Recovery (Deadlock Scenario, Avail $(0,0,0)$) | Deadlocked $\{P_1, P_2, P_3, P_4\}$; WFG Cycle $P_1 \to P_2 \to P_3 \to P_1$; Terminate $P_1$ (holds 2) $\to$ Avail $(2,0,0) \to$ completion $P_0, P_3, P_2, P_4$ | `Deadlocked: {P1,P2,P3,P4}; Cycle: P1->P2->P3->P1; Victim: P1; Avail: (2,0,0)` | **PASS** |
| **TC5** | Deadlock Prevention (Hold & Wait + Circular Wait) | H&W All-or-Nothing completion order $\langle P_0, P_1, P_2, P_3, P_4 \rangle$; Circular Wait ordering flags $P_3$ (holds $C$, requests $A$) | `H&W order <P0,P1,P2,P3,P4>; CW violation on P3: Yes (all other processes comply)` | **PASS** |
| **TC6** | Boundary & Invalid Input Verification (6 edge-cases) | (a) Avail $(0,0,0) \to$ exact unsafe message; (b) $P_3$ Need $(0,0,0) \to$ note shown; (c) $P_9 \to$ invalid ID; (d) negative request $\to$ rejected; (e) Alloc $>$ Max $\to$ blocked; (f) Request $>$ Need $\to$ claim exceeded | `(a)✓ (b)✓ (c)✓ (d)✓ (e)✓ (f)✓` | **PASS** |

---

## How to Run Locally

### Option 1: Direct Double-Click (Zero Setup)
Simply double-click `index.html` in your file explorer. It will immediately open in Google Chrome, Microsoft Edge, Mozilla Firefox, or Safari with full interactivity and local storage persistence.

### Option 2: Local Static Web Server (Optional)
Using Node.js:
```bash
npx serve .
```
Or using Python 3:
```bash
python -m http.server 8080
```
Then visit `http://localhost:8080` in your web browser.

---

## How to Deploy to GitHub Pages in 4 Steps

Deploying DeadlockLens to GitHub Pages requires no build configuration:

1. **Initialize Git Repository:**
   ```bash
   git init
   git add index.html README.md
   git commit -m "feat: initial commit of DeadlockLens university dashboard"
   ```

2. **Create a Remote Repository on GitHub:**
   - Go to [GitHub](https://github.com), click **New repository**, name it `DeadlockLens` (or any name), and leave it empty.
   - Link your local repository to GitHub:
     ```bash
     git remote add origin https://github.com/<your-username>/DeadlockLens.git
     git branch -M main
     git push -u origin main
     ```

3. **Enable GitHub Pages:**
   - In your GitHub repository, navigate to **Settings** &rarr; **Pages** (in the left sidebar).
   - Under **Build and deployment** &gt; **Source**, select **Deploy from a branch**.
   - Under **Branch**, select `main` branch and `/ (root)` folder, then click **Save**.

4. **Access Live URL:**
   - Within 60 seconds, your site will be live at:
     ```
     https://<your-username>.github.io/DeadlockLens/
     ```
   - Double-check that all features, test suites, and presets work directly from the hosted URL.

---

## System Architecture & Design Standards

- **Single-File Integrity:** `index.html` encapsulates the entire design system, algorithm engine, and UI controller without requiring external dependencies.
- **CSS Design Tokens:** Semantic CSS variables for theming (`--bg-base`, `--border-subtle`, `--primary-cyan`, `--safe-green`, `--unsafe-red`, etc.).
- **Pure JavaScript Classes:**
  - `DeadlockAlgorithms`: Stateless pure mathematical functions.
  - `AppState`: Reactive state model with `localStorage` persistence.
  - `UIController`: Event delegation, DOM reconciliation, keyboard handling, SVG graph rendering, and stepper animation player.
- **Accessibility:** ARIA tablists, semantic roles, high contrast ratios ($> 7:1$), keyboard shortcuts (`1`–`7`, `Space`, `ArrowRight`, `ArrowLeft`, `T`, `Esc`), and reduced motion overrides.

---
*Developed for University Operating Systems Activity 2 • Evaluated on 100% Test Case Compliance.*
