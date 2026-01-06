# /apply - Agent-First Application Package Generator

Generate complete job application package using agent-based architecture with mandatory verification gates.

---

## What This Command Does

**HYBRID Architecture**: Interactive command (collects user input) + Application Orchestrator Agent (autonomous execution).

**Generates**:
1. Strategic JD assessment (fit scoring, competency alignment, spinning recommendations)
2. Tailored resume (bullets per USER_PROFILE.md distribution, 240-260 chars each)
3. Cover letter (8-12 lines, 150-200 words, Template 1 Minimalist)
4. Outreach strategy (Track G OR Tracks A-F with 3-tier escalation)

**All files DOCX-converted** (Resume.docx, Coverletter.docx) after verification passes.

---

## Workflow Overview

```
User invokes /apply
    ↓
Command asks clarification questions (PHASE 0)
    ↓
Command spawns Application Orchestrator Agent (passes all context)
    ↓
Orchestrator spawns creator/verifier agents sequentially
    ↓
Each agent creates → verifies → proceeds (or retries once)
    ↓
Command receives final report from orchestrator
    ↓
Command presents results to user
```

---

## PHASE 0: Clarification Questions (Bundled - Single Prompt)

**You are Claude, running this command interactively. Ask ALL questions at once in a single prompt BEFORE spawning orchestrator agent.**

---

**Present this to the user:**

"I'll help you generate a complete application package. Please provide the following information:

**1. Job Source** - Where did you find this job?
   - Newsletter (Lenny's, Reforge, etc.)
   - LinkedIn Post
   - LinkedIn Group
   - Networking Event
   - Job Board / Company Website

**2. Referral Status** - Do you have family/close referrals at this company?
   - YES → I'll skip outreach creation (saves ~60 min)
   - NO → I'll create 6-track outreach strategy

**3. Cover Letter Required?** - Does the job posting require a cover letter?
   - YES → I'll create a tailored cover letter
   - NO → I'll skip cover letter creation (saves ~30 min + tokens)
   - UNSURE → I'll create one anyway (better to have it)

**4. Spinning Strategy** - How should I adapt your resume bullets?
   - Option A: Let JD Assessor Agent recommend (I'll analyze and suggest)
   - Option B: You specify (e.g., 'Spin [Company 2] toward healthcare tech')

**5. Job Description** - Paste the complete JD below:
   (Include company name and role title - I'll extract them automatically)
   (or provide URL - I'll fetch it)

---

**Please answer in this format:**

```
1: [Job Board / Company Website]
2: [YES or NO]
3: [YES, NO, or UNSURE]
4: [Option A or Option B with details]
5: [Full JD text or URL]
```
"

---

**Then parse the user's response:**

Extract values and store:
- **Question 1** → `job_source` (determines track_type: "G" if Newsletter/LinkedIn/Event, "A-F" if Job Board)
- **Question 2** → `has_referral` (boolean: true if YES, false if NO)
- **Question 3** → `requires_cover_letter` (boolean: true if YES or UNSURE, false if NO)
- **Question 4** → `spinning_strategy` (user-specified string or "agent-recommend")
- **Question 5** → `jd_text` (if URL provided, use WebFetch to extract)
  - **Auto-extract**: Parse company name and role title from JD text
  - Look for company name in JD header, "About [Company]" section, or domain context
  - Look for role title in JD header or "Job Title:" field
  - Store as `company_name` and `role_title`

**Conditional Components (determined from answers):**
- **Resume**: ALWAYS created (mandatory)
- **Cover Letter**: Created only if `requires_cover_letter=true`
- **Outreach**: Created only if `has_referral=false`

---

## PHASE 1: Determine Application Folder

Based on role title and JD analysis, determine category:

**Categories**:
- **Data PM Applications**: Role involves data platforms, analytics, data products
- **General PM Applications**: Standard product management roles
- **Others**: Non-PM roles or hybrid roles

**Folder path format**:
```
APPLICATIONS/[Category]/[Company]_[Role]/
```

**Example**: `APPLICATIONS/Data PM Applications/MotherDuck_CorePlatformPM/`

---

## PHASE 2: Sequential Agent Orchestration

**You (Claude, running /apply command) will spawn each agent directly using Task tool and track state.**

**CRITICAL**: Do NOT delegate to orchestrator agent. You must spawn JD Assessor, Resume Creator, Resume Verifier, Cover Letter Creator, Cover Letter Verifier, Outreach Creator, Outreach Verifier agents sequentially with explicit Task tool calls.

---

### STEP 1: Create Folder Structure

Use Bash to create folders (DOCX subfolder = same name as parent, shorten "Product Manager" → "PM"):

```bash
# Example: For Snapsheet Data Product Manager role:
# Parent: APPLICATIONS/Data PM/Snapsheet_DataPM/
# DOCX subfolder: APPLICATIONS/Data PM/Snapsheet_DataPM/Snapsheet_DataPM/
mkdir -p "APPLICATIONS/[Category]/[Company]_[Role]/[Company]_[Role]"
```

Verify folder created before proceeding.

---

### STEP 2: JD Assessment

Present to user: "Creating strategic JD assessment..."

Spawn JD Assessor Agent using Task tool:
```
Task(
  subagent_type: "general-purpose",
  description: "Analyze job description and create JD.md",
  prompt: |
    You are the JD Assessor Agent.

    Read and execute: .claude/agents/jd-assessor.md

    Context:
    - Company: [company_name]
    - Role: [role_title]
    - JD Text: [jd_text]
    - Output Path: APPLICATIONS/[Category]/[Company]_[Role]/JD.md
    - Spinning Strategy: [spinning_strategy]

    Follow the 10-step strategic assessment process.
    Create JD.md with TWO SECTIONS:
    1. Strategic Assessment (fit scoring, competency alignment)
    2. Execution Scaffolding (competency weightage, bullet selection strategy, spinning details)

    Return completion message with fit score and interest level.
)
```

Wait for JD Assessor to complete.

Verify JD.md was created:
```bash
test -f "APPLICATIONS/[Category]/[Company]_[Role]/JD.md" && echo "✅ JD.md exists" || echo "❌ JD.md missing"
```

If JD.md missing → ABORT: "JD Assessment failed. Cannot proceed without strategic assessment."

If JD.md exists → Present to user: "✅ JD Assessment complete"

---

### STEP 3: Resume Creation (ATTEMPT 1)

Present to user: "Creating tailored resume..."

Spawn Resume Creator Agent:
```
Task(
  subagent_type: "general-purpose",
  description: "Create customized resume",
  prompt: |
    You are the Resume Creator Agent.

    STEP 0 - READ MASTER_TEMPLATE.MD FIRST (MANDATORY):
    Read structure template: PM PLAYBOOK/MASTER_TEMPLATE.md
    Study EXACT format before creating resume.

    STEP 1 - READ PLAYBOOKS:
    Read and follow: PM PLAYBOOK/02_RESUME.md (675 lines - READ COMPLETELY)
    Read bullet library: PM PLAYBOOK/MASTER_RESUME.md (lines 51-625)
    Read JD analysis: APPLICATIONS/[Category]/[Company]_[Role]/JD.md

    Spinning Strategy: [spinning_strategy]

    Your task:
    1. Select 13 bullets from MASTER_RESUME.md based on JD.md competency weightage
    2. Apply spinning strategy to mold bullets to target industry
    3. Apply 6-point framework to EVERY bullet
    4. CRITICAL: Verify character count for EACH bullet using Bash tool:
       echo "[bullet text]" | wc -c
       Must be 240-260 characters
    5. Create summary (360-380 chars total, 3 lines max, JD keywords frontloaded, NO metrics)
    6. Create skills section (follow SKILLS SECTION INTELLIGENCE FRAMEWORK from playbook)
    7. Use Write tool to create: APPLICATIONS/[Category]/[Company]_[Role]/RESUME.md

    MANDATORY REQUIREMENTS:
    - Exactly 13 bullets (3-3-3-2-2 distribution)
    - 4 sections (Summary + Professional Experience + Skills + Education)
    - Education section: Include static content from MASTER_TEMPLATE.md (lines 71-75)
    - Clean role titles: "[Company] | [Role] | [Year]-Present" (NO descriptors)
    - All bullets 240-260 chars (VERIFY with wc -c for EACH)
    - NO Certifications section

    Create the file and report completion.
)
```

Wait for Resume Creator to complete.

Present to user: "✅ Resume created (13 bullets)"

---

### STEP 4: Resume Verification (ATTEMPT 1)

Present to user: "Verifying resume quality..."

Spawn Resume Verifier Agent:
```
Task(
  subagent_type: "general-purpose",
  description: "Verify resume meets all requirements",
  prompt: |
    You are the Resume Verifier Agent.

    Read file: APPLICATIONS/[Category]/[Company]_[Role]/RESUME.md

    Verify ALL checks and return structured report:

    1. Character Count Verification (CRITICAL):
       For EACH of the 13 bullets:
       - Extract bullet text
       - Run Bash: echo "[bullet text]" | wc -c
       - Verify: 240 <= count <= 260
       - Record: Bullet X: [count] chars [PASS/FAIL]

    2. Structure Verification:
       - Count sections (must be exactly 4)
       - Count bullets (must be exactly 13)
       - Verify distribution (3-3-3-2-2)
       - Education section present

    3. Quality Verification:
       - Extract action verb from each bullet
       - Verify NO verb repeats
       - Verify metric diversity

    4. Format Verification:
       - Role titles have NO descriptors
       - Summary is 360-380 chars total
       - Education section present (NO Certifications section)

    Return structured report:
    ```
    OVERALL: [PASS/FAIL]

    Character Count Check:
    - Bullet 1 ([Company 1]): 256 chars ✓
    - Bullet 2 ([Company 1]): 248 chars ✗ (TOO SHORT)
    ...

    Section Count: [X] sections [PASS/FAIL]
    Bullet Count: [X] bullets [PASS/FAIL]
    Distribution: [X-X-X-X-X] [PASS/FAIL]
    Action Verb Diversity: [X] unique verbs [PASS/FAIL]
    ```

    If ANY check fails, OVERALL = FAIL.
    If ALL checks pass, OVERALL = PASS.
)
```

Wait for Resume Verifier to complete.

Read verification report and check OVERALL status.

**If OVERALL = PASS:**
- Present to user: "✅ Resume verified (all checks passed)"
- Convert to DOCX (subfolder = parent folder name):
  ```bash
  /convert "APPLICATIONS/[Category]/[Company]_[Role]/RESUME.md" [Company]_[Role]
  ```
- **Route based on conditional flags:**
  - If `requires_cover_letter=true` → Proceed to STEP 7 (Cover Letter Creation)
  - Else if `has_referral=false` → Skip to STEP 11 (Outreach Creation)
  - Else → Skip to STEP 14 (Final Report)

**If OVERALL = FAIL:**
- Present to user: "⚠️ Resume verification failed. Auto-retrying..."
- Show failure reasons from verification report
- Proceed to STEP 5 (Resume Retry)

---

### STEP 5: Resume Bullet-Level Retry (ATTEMPT 2 - TOKEN-OPTIMIZED)

**Only execute if STEP 4 returned OVERALL = FAIL**

**KEY OPTIMIZATION: Only regenerate FAILING bullets, not the entire resume.**

**Parse verifier report to extract failing bullets:**
- Look for "FAILING_BULLETS: [X, Y, Z]" in verifier output
- Extract bullet numbers (1-13) that failed character count or other critical checks
- Store as `failing_bullets` list (e.g., [2, 5, 11])

Present to user: "⚠️ {len(failing_bullets)} bullets failed verification. Regenerating only those bullets..."

**Spawn Resume Creator in PARTIAL REGENERATION mode:**
```
Task(
  subagent_type: "general-purpose",
  description: "Fix failing resume bullets (PARTIAL RETRY)",
  prompt: |
    You are the Resume Creator Agent (PARTIAL REGENERATION MODE).

    **IMPORTANT: Only regenerate the FAILING bullets. Keep all passing content intact.**

    Resume file: APPLICATIONS/[Category]/[Company]_[Role]/RESUME.md

    FAILING BULLETS TO FIX:
    [Insert failing_bullets list with specific issues from verifier]
    Example:
    - Bullet 2 ([Company 1]): 238 chars → ADD 2-12 chars to reach 240-260
    - Bullet 5 ([Company 2]): 267 chars → REMOVE 7-17 chars to reach 240-260
    - Bullet 11 ([Company 4]): 235 chars → ADD 5-15 chars to reach 240-260

    **Your task (MINIMAL EDITS ONLY):**
    1. Read current RESUME.md
    2. For EACH failing bullet:
       a. Extract the current bullet text
       b. Adjust to meet 240-260 character requirement
       c. Preserve the 6-point framework structure
       d. Maintain the same core meaning and metrics
    3. Use Edit tool to replace ONLY the failing bullets (do NOT rewrite entire file)
    4. Run `wc -c` to verify EACH fixed bullet is now 240-260 chars

    **DO NOT:**
    - ❌ Rewrite the entire resume
    - ❌ Change bullets that already passed (bullets not in failing list)
    - ❌ Modify summary, skills, or education sections (unless specifically flagged)

    **DO:**
    - ✅ Only edit the specific failing bullets
    - ✅ Make minimal character adjustments (add/remove a few words)
    - ✅ Verify each fixed bullet with `wc -c` before proceeding
    - ✅ Return list of fixed bullets with new character counts

    Report completion with: "Fixed X bullets: [bullet numbers] - all now 240-260 chars"
)
```

Wait for completion.

**Spawn Resume Verifier for targeted re-verification:**
```
Task(
  subagent_type: "general-purpose",
  description: "Re-verify fixed bullets only",
  prompt: |
    You are the Resume Verifier Agent (TARGETED RE-VERIFICATION).

    Resume file: APPLICATIONS/[Category]/[Company]_[Role]/RESUME.md

    **PREVIOUSLY FAILING BULLETS: [failing_bullets list]**

    Re-verify ONLY these specific bullets:
    - Run `echo "[bullet text]" | wc -c` for each previously failing bullet
    - Confirm each is now 240-260 characters

    Also do a quick full-resume sanity check:
    - Confirm still 13 bullets total
    - Confirm 4 sections present

    Return:
    ```
    RE-VERIFICATION RESULT: [PASS/FAIL]

    Previously Failing Bullets (Re-checked):
    - Bullet X: [new_count] chars [✓ PASS / ✗ STILL FAIL]
    - Bullet Y: [new_count] chars [✓ PASS / ✗ STILL FAIL]

    Sanity Check:
    - Total bullets: 13 ✓
    - Total sections: 4 ✓
    ```
)
```

Wait for re-verification report.

**If RE-VERIFICATION = PASS:**
- Present to user: "✅ Resume verified on retry (fixed {len(failing_bullets)} bullets)"
- Convert to DOCX
- **Route based on conditional flags:**
  - If `requires_cover_letter=true` → Proceed to STEP 7
  - Else if `has_referral=false` → Skip to STEP 11
  - Else → Skip to STEP 14

**If RE-VERIFICATION = FAIL:**
- Present to user: "❌ Resume still has failing bullets after retry"
- Show detailed failure report with specific bullets still failing
- Ask user: "Options: (1) Manual fix (2) Abandon application. Your choice?"
- Wait for user response
- If user chooses (1): Guide them to manually edit RESUME.md, then continue
- If user chooses (2): ABORT workflow and clean up folder

---

### STEP 6: (Reserved - Resume Retry handled in STEP 5)

---

### STEP 7: Cover Letter Creation (ATTEMPT 1)

**CONDITIONAL: Skip this step if `requires_cover_letter=false`. Go directly to STEP 11 (Outreach) or STEP 14 (Final Report) based on referral status.**

**If `requires_cover_letter=true`:**

Present to user: "Creating cover letter..."

Spawn CoverLetter Creator Agent:
```
Task(
  subagent_type: "general-purpose",
  description: "Create cover letter",
  prompt: |
    You are the CoverLetter Creator Agent.

    Read and follow: PM PLAYBOOK/03_COVERLETTER.md (280 lines - READ COMPLETELY FIRST)
    Read JD analysis: APPLICATIONS/[Category]/[Company]_[Role]/JD.md

    Your task:
    1. Research company hook (from JD, news, LinkedIn)
    2. Select strongest relevant achievement with quantified outcome
    3. Write 4-paragraph structure (Hook → Value → Alignment → CTA)
    4. Use Template 1: Minimalist (lines 94-114 in playbook)
    5. Compress to 8-12 lines, 150-200 words (crisp, no fluff)
    6. Use Write tool to create: APPLICATIONS/[Category]/[Company]_[Role]/COVERLETTER.md

    MANDATORY REQUIREMENTS:
    - 8-12 lines total
    - 150-200 words
    - Template 1 Minimalist format
    - No formal headers (no "Re:", no H2 section titles)
    - 4 simple paragraphs: Hook → Value → Alignment → CTA
    - Casual but professional tone

    Create the file and report completion.
)
```

Wait for completion.

Present to user: "✅ Cover letter created"

---

### STEP 8: Cover Letter Verification (ATTEMPT 1)

Present to user: "Verifying cover letter..."

Spawn CoverLetter Verifier Agent:
```
Task(
  subagent_type: "general-purpose",
  description: "Verify cover letter",
  prompt: |
    You are the CoverLetter Verifier Agent.

    Read file: APPLICATIONS/[Category]/[Company]_[Role]/COVERLETTER.md

    Verify ALL checks:

    1. Word Count Verification:
       - Count total words (use Bash: wc -w)
       - Verify: 150 <= count <= 200

    2. Line Count Verification:
       - Count content lines (exclude empty lines)
       - Verify: 8 <= count <= 12

    3. Structure Verification:
       - 4 paragraphs present (Hook, Value, Alignment, CTA)
       - No formal headers ("Re:", H2 titles, etc.)
       - Template 1 format followed

    Return structured report:
    ```
    OVERALL: [PASS/FAIL]

    Word Count: [X] words [PASS/FAIL]
    Line Count: [X] lines [PASS/FAIL]
    Structure: 4 paragraphs [PASS/FAIL]
    No Formal Headers: [PASS/FAIL]
    Template 1 Format: [PASS/FAIL]
    ```

    If ANY check fails, OVERALL = FAIL.
)
```

Wait for verification report.

**If OVERALL = PASS:**
- Present to user: "✅ Cover letter verified"
- Convert to DOCX (subfolder = parent folder name):
  ```bash
  /convert "APPLICATIONS/[Category]/[Company]_[Role]/COVERLETTER.md" [Company]_[Role]
  ```
- Proceed to STEP 11 (Outreach Creation) if has_referral=false, else STEP 14 (Final Report)

**If OVERALL = FAIL:**
- Present to user: "⚠️ Cover letter verification failed. Auto-retrying..."
- Show failure reasons
- Proceed to STEP 9 (Cover Letter Retry)

---

### STEP 9: Cover Letter Creation (ATTEMPT 2 - Auto-Retry)

**Only execute if STEP 8 returned OVERALL = FAIL**

Present to user: "Regenerating cover letter..."

Spawn CoverLetter Creator AGAIN with failure context.
Spawn CoverLetter Verifier AGAIN.

**If OVERALL = PASS:**
- Convert to DOCX
- Proceed to STEP 11 (or STEP 14 if has_referral=true)

**If OVERALL = FAIL:**
- Ask user: "(1) Manual fix (2) Abandon"
- Handle user decision

---

### STEP 10: (Reserved - Cover Letter Retry handled in STEP 9)

---

### STEP 11: Outreach Creation (ATTEMPT 1)

**Skip this step if has_referral=true. Go directly to STEP 14.**

Present to user: "Creating outreach strategy..."

Spawn Outreach Creator Agent:
```
Task(
  subagent_type: "general-purpose",
  description: "Create outreach",
  prompt: |
    You are the Outreach Creator Agent.

    Track Type: [Determine based on job_source: "G" if Newsletter/LinkedIn/Event, "A-F" if Job Board]
    Playbook to use: PM PLAYBOOK/04_OUTREACH.md

    Read and follow playbook (READ COMPLETELY FIRST)
    Read JD analysis: APPLICATIONS/[Category]/[Company]_[Role]/JD.md

    Your task:
    1. Create outreach following exact templates (no deviations)
    2. Create 3-tier escalation for each track (T+0, T+7, T+14 days)
    3. Personalize with company research
    4. Use Write tool to create: APPLICATIONS/[Category]/[Company]_[Role]/OUTREACH.md

    MANDATORY REQUIREMENTS:
    - Correct track type
    - 3-tier escalation for each track
    - Character counts:
      - Initial message: 600-800 chars
      - Escalation 1: 400-600 chars
      - Escalation 2: 300-500 chars
    - Follows exact templates from playbook

    Create the file and report completion.
)
```

Wait for completion.

Present to user: "✅ Outreach strategy created"

---

### STEP 12: Outreach Verification (ATTEMPT 1)

**Skip if has_referral=true**

Present to user: "Verifying outreach..."

Spawn Outreach Verifier Agent:
```
Task(
  subagent_type: "general-purpose",
  description: "Verify outreach",
  prompt: |
    You are the Outreach Verifier Agent.

    Read file: APPLICATIONS/[Category]/[Company]_[Role]/OUTREACH.md
    Expected track type: [Track Type from STEP 11]

    Verify ALL checks:

    1. Track Type Verification:
       - Verify correct track type is present

    2. Escalation Verification:
       - Verify 3-tier escalation exists (T+0, T+7, T+14)
       - Count messages per tier

    3. Character Count Verification:
       - Initial message: 600-800 chars
       - Escalation 1: 400-600 chars
       - Escalation 2: 300-500 chars

    Return structured report:
    ```
    OVERALL: [PASS/FAIL]

    Track Type: [Found] vs [Expected] [PASS/FAIL]
    Escalation Structure: 3 tiers present [PASS/FAIL]
    Character Counts:
      - Initial: [X] chars [PASS/FAIL]
      - Escalation 1: [X] chars [PASS/FAIL]
      - Escalation 2: [X] chars [PASS/FAIL]
    ```
)
```

Wait for verification.

**If OVERALL = PASS:**
- Present to user: "✅ Outreach verified"
- Proceed to STEP 14 (Final Report)

**If OVERALL = FAIL:**
- Auto-retry (STEP 13)

---

### STEP 13: Outreach Retry (ATTEMPT 2)

**Only if STEP 12 failed**

Spawn Outreach Creator AGAIN.
Spawn Outreach Verifier AGAIN.

**If PASS:** Proceed to STEP 14
**If FAIL:** Ask user decision

---

### STEP 14: Final Report

Present comprehensive summary to user based on what was created:

```
✅ APPLICATION PACKAGE COMPLETE

Company: [company_name]
Role: [role_title]
Folder: APPLICATIONS/[Category]/[Company]_[Role]/

Files Created:
1. JD.md - Strategic assessment + execution scaffolding ✓

2. RESUME.md → Resume.docx (VERIFIED ✓)
   - 13 bullets (3-3-3-2-2 distribution)
   - All bullets 240-260 characters

3. COVERLETTER.md → Coverletter.docx
   [If requires_cover_letter=true]: (VERIFIED ✓) - 8-12 lines, 150-200 words
   [If requires_cover_letter=false]: ⏭️ SKIPPED (not required by job posting)

4. OUTREACH.md
   [If has_referral=false]: (VERIFIED ✓) - Track [Type] with 3-tier escalation
   [If has_referral=true]: ⏭️ SKIPPED (family/close referral available)

All DOCX files are in [Company]_[Role]/ subfolder - drag-and-drop ready for Applied folder.

Token Savings Summary:
- Cover Letter: [CREATED / SKIPPED (~30 min saved)]
- Outreach: [CREATED / SKIPPED (~60 min saved)]
```

---

## PHASE 3: Complete

Application package generation is now complete. All agents were spawned sequentially, all verification gates were enforced, and all files are ready for submission

## Notes

- **This command does direct orchestration** - you spawn each agent sequentially
- **Explicit Task tool calls guarantee agent spawning** - no delegation to orchestrator
- **Verification gates are enforced** - PASS/FAIL blocks proceeding to next step
- **1 auto-retry per component** - if fails twice, ask user (manual fix or abandon)
- **DOCX conversion only after verification passes** - prevents wasted conversions
- **Real-time feedback** - user sees progress after each step completes
