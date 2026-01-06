# /init - System Setup & Validation

Initialize and validate the job application automation system.

---

## What This Command Does

1. **Validates dependencies** (python3, python-docx)
2. **Checks folder structure** (APPLICATIONS/, YOUR_PROFILE/, PLAYBOOK/)
3. **Verifies core files** (.claude/agents/, .claude/commands/)
4. **Checks user profile** (USER_PROFILE.md, USER_BULLETS.md)
5. **Reports setup status**

---

## Execution Steps

### Step 1: Check Python Dependencies

```bash
# Check Python 3 installed
python3 --version

# Check python-docx library
python3 -c "import docx; print('python-docx installed')" 2>/dev/null || echo "python-docx NOT installed"
```

**If python-docx missing:**
```bash
pip3 install python-docx
```

---

### Step 2: Verify Folder Structure

Check these folders exist:
- `APPLICATIONS/`
- `YOUR_PROFILE/`
- `YOUR_PROFILE/examples/`
- `PLAYBOOK/`
- `.claude/commands/`
- `.claude/agents/`

```bash
# Verify folders
test -d "APPLICATIONS" && echo "APPLICATIONS/ exists" || echo "APPLICATIONS/ MISSING"
test -d "YOUR_PROFILE" && echo "YOUR_PROFILE/ exists" || echo "YOUR_PROFILE/ MISSING"
test -d "YOUR_PROFILE/examples" && echo "YOUR_PROFILE/examples/ exists" || echo "YOUR_PROFILE/examples/ MISSING"
test -d "PLAYBOOK" && echo "PLAYBOOK/ exists" || echo "PLAYBOOK/ MISSING"
test -d ".claude/commands" && echo ".claude/commands/ exists" || echo ".claude/commands/ MISSING"
test -d ".claude/agents" && echo ".claude/agents/ exists" || echo ".claude/agents/ MISSING"
```

---

### Step 3: Verify Core Files

**.claude/commands/ (2 files):**
- [x] `.claude/commands/apply.md`
- [x] `.claude/commands/init.md`

**.claude/agents/ (8 files):**
- [x] `.claude/agents/application-orchestrator.md`
- [x] `.claude/agents/jd-assessor.md`
- [x] `.claude/agents/resume-creator.md`
- [x] `.claude/agents/resume-verifier.md`
- [x] `.claude/agents/coverletter-creator.md`
- [x] `.claude/agents/coverletter-verifier.md`
- [x] `.claude/agents/outreach-creator.md`
- [x] `.claude/agents/outreach-verifier.md`

**PLAYBOOK/ (6 files):**
- [x] `PLAYBOOK/MASTER_TEMPLATE.md`
- [x] `PLAYBOOK/MASTER_RESUME.md`
- [x] `PLAYBOOK/RESUME_FRAMEWORK.md`
- [x] `PLAYBOOK/COVERLETTER_FRAMEWORK.md`
- [x] `PLAYBOOK/OUTREACH_FRAMEWORK.md`
- [x] `PLAYBOOK/resume_generator.py`

**YOUR_PROFILE/ (2 user files + 3 examples):**
- [x] `YOUR_PROFILE/USER_PROFILE.md`
- [x] `YOUR_PROFILE/USER_BULLETS.md`
- [x] `YOUR_PROFILE/examples/EXAMPLE_USER_PROFILE.md`
- [x] `YOUR_PROFILE/examples/EXAMPLE_USER_BULLETS.md`
- [x] `YOUR_PROFILE/examples/EXAMPLE_JD.md`

**Root files:**
- [x] `CLAUDE.md`
- [x] `README.md`

```bash
# Verify critical files
test -f "CLAUDE.md" && echo "CLAUDE.md exists" || echo "CLAUDE.md MISSING"
test -f "PLAYBOOK/resume_generator.py" && echo "resume_generator.py exists" || echo "resume_generator.py MISSING"
test -f "YOUR_PROFILE/USER_PROFILE.md" && echo "USER_PROFILE.md exists" || echo "USER_PROFILE.md MISSING"
test -f "YOUR_PROFILE/USER_BULLETS.md" && echo "USER_BULLETS.md exists" || echo "USER_BULLETS.md MISSING"
```

---

### Step 4: Check User Profile Status

Verify user has filled in their profile:

```bash
# Check if USER_PROFILE.md has been customized (not just template)
grep -q "YOUR NAME HERE\|FILL THIS\|TODO" "YOUR_PROFILE/USER_PROFILE.md" && echo "USER_PROFILE.md needs to be filled in" || echo "USER_PROFILE.md appears customized"

# Check if USER_BULLETS.md has content
BULLET_COUNT=$(grep -c "^- " "YOUR_PROFILE/USER_BULLETS.md" 2>/dev/null || echo "0")
echo "USER_BULLETS.md has $BULLET_COUNT bullets (recommend 40-60)"
```

---

### Step 5: Count Agent Files

```bash
# Count agents
AGENT_COUNT=$(ls -1 .claude/agents/*.md 2>/dev/null | wc -l | tr -d ' ')
echo "Found $AGENT_COUNT agent files (expected 8)"
```

---

## Success Report

If all checks pass, you should see:

```
================================================================================
SYSTEM INITIALIZATION COMPLETE
================================================================================

Dependencies:
   - Python 3: Installed
   - python-docx: Installed

Folder Structure:
   - APPLICATIONS/
   - YOUR_PROFILE/
   - YOUR_PROFILE/examples/
   - PLAYBOOK/
   - .claude/commands/
   - .claude/agents/

Core Files:
   - .claude/ (10 files: 2 commands + 8 agents)
   - PLAYBOOK/ (6 files)
   - YOUR_PROFILE/ (5 files: 2 user + 3 examples)
   - CLAUDE.md
   - README.md

User Profile Status:
   - USER_PROFILE.md: [Filled/Needs attention]
   - USER_BULLETS.md: [X bullets found]

System Health: ALL CHECKS PASSED

================================================================================
READY TO USE
================================================================================

Next steps:
1. Fill in YOUR_PROFILE/USER_PROFILE.md with your information
2. Fill in YOUR_PROFILE/USER_BULLETS.md with your accomplishments (40-60 bullets)
3. See YOUR_PROFILE/examples/ for reference
4. Run /apply with a job description

================================================================================
```

---

## Troubleshooting

### Issue: python-docx not found
**Fix:** `pip3 install python-docx`

### Issue: USER_PROFILE.md not found
**Fix:** Create `YOUR_PROFILE/USER_PROFILE.md` using the example as reference

### Issue: No bullets found
**Fix:** Add your accomplishments to `YOUR_PROFILE/USER_BULLETS.md` (40-60 bullets recommended)

### Issue: Missing agent files
**Check:** Ensure all 8 agent files exist in `.claude/agents/`

---

## Notes

- This command is **READ-ONLY** - it doesn't modify any files
- Run `/init` anytime to verify system health
- Safe to run multiple times
