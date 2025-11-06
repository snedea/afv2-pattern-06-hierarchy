# AFv2 Pattern #6: Hierarchy - Test Cases

## Overview

**Pattern:** Supervisor-orchestrated task delegation to specialist roles with review gates
**Flow:** Start → Supervisor → Checker → [Worker → Reviewer → loop back to Checker] → Final → Direct Reply
**Repository:** https://github.com/snedea/afv2-pattern-06-hierarchy

---

## Prerequisites

### 1. Import Pattern into Flowise

1. Open Flowise UI (http://localhost:3000)
2. Navigate to **Agentflows** section
3. Click **"Add New"** → **"Import Agentflow"**
4. Upload `06-hierarchy.json`
5. Pattern should load with nodes:
   - Start
   - Agent.Supervisor
   - Agent.Checker
   - Agent.Worker
   - Agent.Reviewer
   - Agent.Final
   - Direct Reply
   - Sticky Notes (documentation)

### 2. Configure API Keys

**All agents require Anthropic API key configuration:**

1. Configure model for all agents:
   - **Credential:** Select your "Anthropic API Key" credential
   - **Model Name:** `claude-sonnet-4-5-20250929`
   - **Temperature:**
     - Supervisor: `0.3` (planning)
     - Checker: `0.1` (deterministic)
     - Worker: `0.4` (creative)
     - Reviewer: `0.2` (analytical)
     - Final: `0.2` (synthesis)
   - **Streaming:** `true`

2. Save the workflow

### 3. Verify Node Configuration

**Expected Configuration:**

| Node | Type | Tools | State Updates | Key Feature |
|------|------|-------|---------------|-------------|
| Agent.Supervisor | Agent | `currentDateTime` | `hierarchy.plan`, `hierarchy.total_steps` | Delegates work with plan |
| Agent.Checker | Agent | N/A | `hierarchy.current_step`, `hierarchy.ready` | Checks if all steps complete |
| Agent.Worker | Agent | `calculator`, `currentDateTime` | `hierarchy.step_result`, `hierarchy.step_status` | Executes delegated work |
| Agent.Reviewer | Agent | `currentDateTime` | `hierarchy.review_status`, `hierarchy.current_step++` | Reviews work quality |
| Agent.Final | Agent | `currentDateTime` | `output` | Aggregates all results |
| Direct Reply | DirectReply | N/A | N/A | Terminal node |

**Role Hierarchy:**
- **Supervisor:** High-level planning and delegation
- **Checker:** Progress tracking and completion detection
- **Worker:** Task execution (one step at a time)
- **Reviewer:** Quality assurance with approval/rejection
- **Final:** Results synthesis

---

## Test Cases

### TC-6.1: Basic Hierarchical Workflow (3 Steps)

**Objective:** Verify supervisor delegates work to worker, reviewer approves, and cycle repeats for all steps

**Input:**
```
Plan and execute a customer onboarding process: 1) Send welcome email, 2) Create user account, 3) Schedule training session.
```

**Expected Execution Flow:**

1. **Agent.Supervisor Executes:**
   - Analyzes input
   - Creates delegation plan with 3 steps:
     ```json
     {
       "step_1": "Send welcome email to customer",
       "step_2": "Create user account with credentials",
       "step_3": "Schedule onboarding training session"
     }
     ```
   - Updates state:
     ```json
     {
       "hierarchy.plan": "[delegation plan]",
       "hierarchy.total_steps": 3,
       "hierarchy.current_step": 0
     }
     ```

2. **Agent.Checker Executes (Iteration 1):**
   - Reads `hierarchy.current_step = 0`
   - Reads `hierarchy.total_steps = 3`
   - Determines: 0 < 3 → More work to do
   - Selects next step: Step 1
   - Updates state:
     ```json
     {
       "hierarchy.current_step": 1,
       "hierarchy.ready": "false"
     }
     ```
   - Routes to Worker

3. **Agent.Worker Executes (Step 1):**
   - Reads `hierarchy.plan` step 1: "Send welcome email"
   - Executes task using tools
   - Produces result: "Welcome email sent to customer@example.com"
   - Updates state:
     ```json
     {
       "hierarchy.step_result": "[email sent confirmation]",
       "hierarchy.step_status": "completed"
     }
     ```

4. **Agent.Reviewer Executes (Review Step 1):**
   - Reads `hierarchy.step_result` from Worker
   - Reviews quality: Email template used, recipient correct, tracking enabled
   - Decision: **APPROVED** ✅
   - Updates state:
     ```json
     {
       "hierarchy.review_status": "approved",
       "hierarchy.current_step": 1  // Increments step counter
     }
     ```
   - Routes back to Checker (loop-back edge)

5. **Agent.Checker Executes (Iteration 2):**
   - Reads `hierarchy.current_step = 1`
   - Determines: 1 < 3 → More work to do
   - Selects next step: Step 2
   - Routes to Worker

6. **Agent.Worker Executes (Step 2):**
   - Reads plan step 2: "Create user account"
   - Executes task
   - Updates state: `hierarchy.step_result`, `hierarchy.step_status`

7. **Agent.Reviewer (Review Step 2):**
   - Reviews and approves
   - Increments: `hierarchy.current_step = 2`
   - Routes back to Checker

8. **Agent.Checker Executes (Iteration 3):**
   - Reads `hierarchy.current_step = 2`
   - Determines: 2 < 3 → More work to do
   - Selects next step: Step 3
   - Routes to Worker

9. **Agent.Worker Executes (Step 3):**
   - Reads plan step 3: "Schedule training session"
   - Executes task
   - Updates state

10. **Agent.Reviewer (Review Step 3):**
    - Reviews and approves
    - Increments: `hierarchy.current_step = 3`
    - Routes back to Checker

11. **Agent.Checker Executes (Final Check):**
    - Reads `hierarchy.current_step = 3`
    - Reads `hierarchy.total_steps = 3`
    - Determines: 3 ≥ 3 → **All steps complete** ✅
    - Updates state: `hierarchy.ready = "true"`
    - Routes to Final agent

12. **Agent.Final Executes:**
    - Aggregates all step results
    - Generates summary report
    - Updates state: `output`

13. **Direct Reply Returns:**
    - Displays completion message with all steps

**Validation Checklist:**

- [ ] **Supervisor Planning:**
  - Created plan with 3 distinct steps
  - State contains `hierarchy.total_steps=3`
  - Initial `hierarchy.current_step=0`

- [ ] **Checker Iteration Tracking:**
  - Checker executed 4 times (iterations 1-3 + final check)
  - Each iteration incremented step correctly
  - Final iteration detected completion (3 ≥ 3)

- [ ] **Worker Execution:**
  - Worker executed exactly 3 times (once per step)
  - Each execution handled different step from plan
  - Worker accessed plan for task details

- [ ] **Reviewer Quality Gates:**
  - Reviewer executed 3 times (once per step)
  - Each review approved work
  - Reviewer incremented `hierarchy.current_step` each time

- [ ] **Loop-Back Behavior:**
  - Reviewer routed back to Checker after each step
  - Loop continued until all steps complete
  - Loop exited when `current_step ≥ total_steps`

- [ ] **Final Aggregation:**
  - Final agent executed ONLY after all steps complete
  - Aggregated results from all 3 steps
  - Generated comprehensive summary

- [ ] **Step Progression:**
  ```
  Iteration 1: current_step 0→1 (Step 1 complete)
  Iteration 2: current_step 1→2 (Step 2 complete)
  Iteration 3: current_step 2→3 (Step 3 complete)
  Final: current_step=3, total=3 → Exit loop
  ```

**Success Criteria:**
- All 3 steps executed in sequence
- Worker executed exactly 3 times
- Reviewer approved all steps
- Loop exited cleanly after completion
- Final report aggregated all results

---

### TC-6.2: Reviewer Rejection and Rework

**Objective:** Verify reviewer can reject work and send back for revision

**Input:**
```
Prepare quarterly financial report: 1) Compile revenue data, 2) Calculate profit margins, 3) Generate executive summary.
```

**Expected Execution Flow:**

1. **Supervisor → Checker → Worker (Step 1):**
   - Worker compiles revenue data
   - Result: "$1.2M revenue (Q3 2024)"
   - Updates state: `hierarchy.step_result`, `hierarchy.step_status="completed"`

2. **Agent.Reviewer (Review Step 1 - FIRST ATTEMPT):**
   - Reviews revenue data
   - Identifies issue: "Missing breakdown by product line"
   - Decision: **REJECTED** ❌
   - Updates state:
     ```json
     {
       "hierarchy.review_status": "rejected",
       "hierarchy.review_feedback": "Revenue data missing product line breakdown. Please add: Product A, Product B, Product C revenue details.",
       "hierarchy.current_step": 1  // Does NOT increment (rework needed)
     }
     ```
   - Routes back to Worker (NOT Checker)

3. **Agent.Worker (Step 1 - REWORK):**
   - Reads `hierarchy.review_feedback`
   - Revises revenue data with product breakdown:
     ```
     Total Revenue: $1.2M (Q3 2024)
     - Product A: $600K (50%)
     - Product B: $400K (33%)
     - Product C: $200K (17%)
     ```
   - Updates state: `hierarchy.step_result` (revised)

4. **Agent.Reviewer (Review Step 1 - SECOND ATTEMPT):**
   - Reviews revised data
   - Confirms product breakdown included
   - Decision: **APPROVED** ✅
   - Updates state:
     ```json
     {
       "hierarchy.review_status": "approved",
       "hierarchy.current_step": 2  // NOW increments
     }
     ```
   - Routes back to Checker

5. **Checker → Worker (Step 2) → Reviewer → ... (continues normally)**

**Validation Checklist:**

- [ ] **Rejection Detection:**
  - Reviewer identified quality issue
  - Set `hierarchy.review_status="rejected"`
  - Provided specific feedback in `hierarchy.review_feedback`

- [ ] **Rework Routing:**
  - Reviewer routed back to Worker (NOT Checker)
  - Worker executed again for same step
  - **CRITICAL:** `hierarchy.current_step` did NOT increment on rejection

- [ ] **Feedback Application:**
  - Worker read `hierarchy.review_feedback` from state
  - Revised output addressed reviewer feedback
  - Second attempt incorporated requested changes

- [ ] **Re-Review:**
  - Reviewer re-evaluated revised work
  - Approved on second attempt
  - **NOW** incremented `hierarchy.current_step`

- [ ] **Continuation:**
  - After approval, flow continued to next step
  - Subsequent steps proceeded normally
  - Final aggregation included revised Step 1 result

**Success Criteria:**
- Reviewer successfully rejected work
- Worker performed rework based on feedback
- Revised work approved on second attempt
- Step counter only incremented after approval
- All steps completed despite rejection

**Rejection Flow Validation:**
```
Step 1 (Attempt 1):
  Worker → Result A (incomplete)
  Reviewer → REJECTED → back to Worker (current_step stays 1)

Step 1 (Attempt 2):
  Worker → Result B (revised, complete)
  Reviewer → APPROVED → to Checker (current_step → 2)

Step 2:
  (continues normally)
```

---

### TC-6.3: Complex Multi-Role Workflow (5 Steps)

**Objective:** Verify pattern scales to more complex workflows with multiple iterations

**Input:**
```
Execute software release process: 1) Run test suite, 2) Build production artifacts, 3) Update documentation, 4) Deploy to staging, 5) Verify deployment.
```

**Expected Execution Flow:**

1. **Supervisor:**
   - Creates 5-step plan
   - Updates state: `hierarchy.total_steps=5`, `hierarchy.current_step=0`

2. **Loop Iterations:**
   ```
   Iteration 1: current_step 0→1 (Step 1: Tests)
   Iteration 2: current_step 1→2 (Step 2: Build)
   Iteration 3: current_step 2→3 (Step 3: Docs)
   Iteration 4: current_step 3→4 (Step 4: Deploy)
   Iteration 5: current_step 4→5 (Step 5: Verify)
   Final: current_step=5, total=5 → Complete
   ```

3. **Each Iteration:**
   - Checker → Worker → Reviewer → back to Checker
   - Worker uses different tools per step:
     - Step 1: Test execution tools
     - Step 2: Build tools
     - Step 3: Documentation tools
     - Step 4: Deployment tools
     - Step 5: Verification tools

4. **Final Aggregation:**
   - Final agent summarizes all 5 steps
   - Release report generated

**Validation Checklist:**

- [ ] **Scalability:**
  - Pattern handled 5 steps successfully
  - Checker iterated 6 times (5 steps + 1 final check)
  - Worker executed 5 times
  - Reviewer executed 5 times

- [ ] **Step Progression:**
  - current_step incremented correctly: 0→1→2→3→4→5
  - Each step executed in correct order
  - No steps skipped

- [ ] **Tool Specialization:**
  - Worker used appropriate tools for each step type
  - Different tools per step (tests, build, docs, deploy, verify)

- [ ] **Completion Detection:**
  - Checker correctly detected completion (5 ≥ 5)
  - Routed to Final agent (not Worker)
  - Loop exited cleanly

- [ ] **Result Aggregation:**
  - Final agent accessed results from all 5 steps
  - Report included complete workflow summary
  - All steps represented in output

**Success Criteria:**
- All 5 steps completed successfully
- Worker executed exactly 5 times (no extra iterations)
- Checker correctly tracked progression
- Final report comprehensive

---

## Common Issues & Debugging

### Issue 1: Infinite Loop (Checker Never Exits)

**Symptoms:**
- Worker executes indefinitely
- Checker never routes to Final agent
- `hierarchy.current_step` not incrementing

**Debugging Steps:**
1. Verify Reviewer increments `hierarchy.current_step`:
   - Should increment after approval
   - Check state update: `hierarchy.current_step = {{ hierarchy.current_step + 1 }}`

2. Check Checker completion condition:
   - Should compare `current_step ≥ total_steps`
   - Should route to Final when condition met

3. Verify loop-back edge:
   - Reviewer should connect back to Checker
   - Checker should have 2 output paths: Worker (more work) | Final (complete)

**Solution:**
- Ensure Reviewer increments step counter
- Configure Checker to detect completion
- Verify edge connections for loop and exit paths

---

### Issue 2: Reviewer Always Approves (No Rejections)

**Symptoms:**
- All work approved regardless of quality
- No rework occurs
- Reviewer doesn't provide meaningful feedback

**Debugging Steps:**
1. Check Reviewer prompt includes quality criteria:
   - Should specify approval standards
   - Should provide rejection criteria
   - Should generate specific feedback

2. Verify Reviewer can route to Worker (rejection path):
   - Should have edge: Reviewer → Worker (for rework)
   - Should have edge: Reviewer → Checker (for approval)

3. Check `hierarchy.review_status` state update:
   - Should be "approved" or "rejected" based on review
   - Should persist rejection feedback

**Solution:**
- Update Reviewer prompt with quality standards
- Add rejection path edge (Reviewer → Worker)
- Configure state updates for rejection feedback

---

### Issue 3: Worker Executes All Steps at Once

**Symptoms:**
- Worker completes all 3 steps in single execution
- No iteration between steps
- Checker doesn't control step-by-step progression

**Debugging Steps:**
1. Verify Worker prompt limits scope:
   - Should execute ONLY current step
   - Should read `hierarchy.current_step` to determine which step
   - Should NOT execute all steps at once

2. Check Checker dispatches one step at a time:
   - Should select next step based on `current_step`
   - Should provide only relevant step details to Worker

3. Verify loop structure:
   - Should be: Checker → Worker → Reviewer → Checker (repeat)
   - NOT: Checker → Worker (all steps) → Reviewer → Final

**Solution:**
- Update Worker prompt to execute single step only
- Configure Checker to dispatch steps individually
- Verify loop iterates for each step

---

### Issue 4: Final Agent Executes Before All Steps Complete

**Symptoms:**
- Final aggregation happens after Step 1
- Steps 2-3 never execute
- Incomplete results

**Debugging Steps:**
1. Check Checker completion condition:
   - Should be `current_step ≥ total_steps`
   - Should NOT exit early (e.g., after Step 1)

2. Verify `hierarchy.current_step` increments correctly:
   - Should be 0 initially
   - Should increment after each approved step
   - Should reach `total_steps` before exiting

3. Check Checker edge connections:
   - Should route to Worker when `current_step < total_steps`
   - Should route to Final when `current_step ≥ total_steps`

**Solution:**
- Fix Checker completion condition
- Ensure step counter increments properly
- Verify edge routing logic

---

### Issue 5: Supervisor Doesn't Create Multi-Step Plan

**Symptoms:**
- Only 1 step in plan
- `hierarchy.total_steps = 1` for multi-step task
- No delegation occurs

**Debugging Steps:**
1. Check Supervisor prompt includes planning instructions:
   - Should break down task into steps
   - Should specify number of steps
   - Should assign work to Worker

2. Verify Supervisor updates state correctly:
   - Should set `hierarchy.plan` with all steps
   - Should set `hierarchy.total_steps` accurately

3. Review Supervisor output:
   - Should produce structured plan (JSON or list)
   - Each step should be actionable

**Solution:**
- Update Supervisor prompt with delegation instructions
- Ensure state updates include full plan
- Test with explicit multi-step task

---

## Test Execution Report

**Test Date:** `[YYYY-MM-DD]`
**Flowise Version:** `[version]`
**Pattern Version:** `1.0`

### Test Results Summary

| Test Case | Status | Total Steps | Rejections | Notes |
|-----------|--------|-------------|------------|-------|
| TC-6.1: Basic 3-Step Workflow | ⬜ Pass / ⬜ Fail | `[3]` | `[0]` | |
| TC-6.2: Rejection & Rework | ⬜ Pass / ⬜ Fail | `[3]` | `[1]` | |
| TC-6.3: Complex 5-Step Workflow | ⬜ Pass / ⬜ Fail | `[5]` | `[0]` | |

### Configuration Details

- **Anthropic Model:** `claude-sonnet-4-5-20250929`
- **Temperature (Supervisor):** `0.3`
- **Temperature (Checker):** `0.1`
- **Temperature (Worker):** `0.4`
- **Temperature (Reviewer):** `0.2`
- **Temperature (Final):** `0.2`
- **Streaming:** `Enabled`
- **Memory Enabled:** `Yes`

### Hierarchy Performance Metrics

| Test Case | Checker Iterations | Worker Executions | Reviewer Approvals | Reviewer Rejections |
|-----------|-------------------|-------------------|-------------------|---------------------|
| TC-6.1 | 4 (3 + 1 final) | 3 | 3 | 0 |
| TC-6.2 | 4 | 4 (1 rework) | 3 | 1 |
| TC-6.3 | 6 (5 + 1 final) | 5 | 5 | 0 |

### Issues Encountered

`[List any issues encountered during testing]`

### Additional Notes

`[Any additional observations or recommendations]`

---

## Pattern Validation

This pattern has been validated against `validate_workflow.py` with the following checks:

✅ **Structural Validation:**
- Proper JSON structure
- All required node and edge fields present

✅ **Node Type Validation:**
- Start node present
- 5 role-based Agent nodes (Supervisor, Checker, Worker, Reviewer, Final)
- 1 DirectReply terminal node
- Sticky Notes for documentation

✅ **Role Hierarchy:**
- Supervisor: Planning and delegation
- Checker: Progress tracking and completion detection
- Worker: Task execution (iterative)
- Reviewer: Quality assurance with approval/rejection
- Final: Results aggregation

✅ **Loop Logic:**
- Loop-back edge exists (Reviewer → Checker)
- Step iterator increments correctly
- Completion detected when `current_step ≥ total_steps`
- Rejection path allows rework (Reviewer → Worker)

✅ **State Management:**
- All agents have memory enabled
- Step counter tracked: `hierarchy.current_step`
- Total steps defined: `hierarchy.total_steps`
- Review status tracked for approvals/rejections
- Plan persisted across iterations

✅ **Tool Specialization:**
- Worker has execution tools (calculator, etc.)
- Other roles have observation/planning tools
- Tool ACLs enforce role boundaries

✅ **Safety Validation:**
- No infinite loop (completion detection enforced)
- Rework path allows quality improvement
- Final aggregation only after all steps complete

---

## Additional Resources

- **Pattern Library:** [Context Foundry AFv2 Patterns](https://github.com/context-foundry/context-foundry/tree/main/extensions/flowise/templates/afv2-patterns)
- **Pattern #6 Repository:** https://github.com/snedea/afv2-pattern-06-hierarchy
- **Flowise Documentation:** https://docs.flowiseai.com/
- **AgentFlow v2 Specification:** [AFv2 Docs](https://docs.flowiseai.com/agentflows)

---

🤖 Built with Context Foundry
