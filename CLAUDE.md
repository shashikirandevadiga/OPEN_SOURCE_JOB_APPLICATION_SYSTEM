# CLAUDE.md - Job Application Automation System

This file provides guidance to Claude Code when working with this repository.

---

## What This System Does

An 8-agent automation system that transforms job descriptions into complete application packages:
- **Input**: Job description + your profile
- **Output**: Tailored resume, cover letter, outreach strategy, ready-to-send DOCX files

---

## Quick Start (3 Steps)

### 1. Set Up Your Profile (2 min)
- **YOUR_PROFILE/USER_PROFILE.md** - Fill with your contact info, work history, education, resume distribution
- **YOUR_PROFILE/USER_BULLETS.md** - Add 40-60 accomplishment bullets (each 240-260 chars)
- See `examples/` folder for reference

### 2. Run the System (1 min)
```bash
claude
/apply
# Paste job description when prompted
```

### 3. Get Your Materials (Automatic)
Output in `APPLICATIONS/[Company]_[Role]/`:
- `JD.md` - Fit score and execution strategy
- `RESUME.md` - 13 tailored bullets (240-260 chars each)
- `COVERLETTER.md` - 4-paragraph cover letter
- `OUTREACH.md` - 6-track outreach strategy
- `[Company]_[Role]/*.docx` - Ready to submit

---

## Slash Commands

### `/apply` - Complete Application Package
**When**: You have a job description ready
**Output**: JD.md, RESUME.md, COVERLETTER.md, OUTREACH.md + DOCX files

### `/init` - System Validation
**When**: First time setup, troubleshooting
**Output**: Validates dependencies, checks file structure

---

## System Architecture

### Agents (8 total)
1. **JD Assessor** - Analyzes JD, scores fit, recommends spinning strategy
2. **Resume Creator** - Selects bullets, applies spinning, creates resume
3. **Resume Verifier** - Validates character counts, structure, quality
4. **CoverLetter Creator** - Creates 4-paragraph minimalist cover letter
5. **CoverLetter Verifier** - Validates word count, format
6. **Outreach Creator** - Creates multi-track outreach with 3-tier escalation
7. **Outreach Verifier** - Validates message quality, personalization
8. **Application Orchestrator** - Coordinates all agents, handles retries

### Key Files
- `YOUR_PROFILE/USER_PROFILE.md` - Your professional profile (YOU fill this)
- `YOUR_PROFILE/USER_BULLETS.md` - Your bullet library (YOU fill this)
- `YOUR_PROFILE/examples/` - Reference examples for profile and bullets
- `PLAYBOOK/MASTER_TEMPLATE.md` - Resume format template
- `PLAYBOOK/MASTER_RESUME.md` - Example bullets (for reference)
- `PLAYBOOK/RESUME_FRAMEWORK.md` - Resume creation rules
- `PLAYBOOK/COVERLETTER_FRAMEWORK.md` - Cover letter templates
- `PLAYBOOK/OUTREACH_FRAMEWORK.md` - Outreach strategy guide

---

## Critical Rules

### Resume (4 Sections Only)
- **Summary**: 360-380 chars, JD keywords frontloaded, NO metrics
- **Professional Experience**: Exactly 13 bullets (3-3-3-2-2 per YOUR_PROFILE.md)
- **Each bullet**: 240-260 characters, 6-point framework
- **Skills**: 3-5 categories, hard skills only, JD-aligned
- **Education**: Static content from YOUR_PROFILE.md
- **NO Certifications section**

### Bullet Format (6-Point Framework)
Each bullet must include:
1. **Action** - Strong verb
2. **Context** - Where/what/who
3. **Method** - How you did it
4. **Result** - Quantified outcome (metric)
5. **Impact** - Business effect
6. **Business Outcome** - Strategic value (revenue ↑, cost ↓, efficiency ↑, retention ↑, or scaling)

### Cover Letter (Template 1 Minimalist)
- **8-12 lines, 150-200 words** (body only, no signature line)
- **4 paragraphs**: Hook → Value → Alignment → CTA
- **NO formal headers** (no "Dear Hiring Manager", no "Re:")
- Casual but professional tone

### Metric Diversification
Use 5 metric types across 13 bullets (no format repeats more than once):
- **TIME**: How much faster? (45 min → 18 min, days/hours saved)
- **VOLUME**: Scale and users (500K+, 60M+ transactions)
- **FREQUENCY**: Recurrence (15+ interviews per cycle)
- **SCOPE**: Geographic reach (9 markets, Fortune 500 clients)
- **QUALITY**: Performance/satisfaction (95% UAT, 96% retention)

---

## File Structure

```
OPEN_SOURCE_JOB_APPLICATION_SYSTEM/
├── .claude/
│   ├── commands/
│   │   ├── apply.md               # /apply command
│   │   └── init.md                # /init command
│   └── agents/
│       ├── application-orchestrator.md
│       ├── jd-assessor.md
│       ├── resume-creator.md
│       ├── resume-verifier.md
│       ├── coverletter-creator.md
│       ├── coverletter-verifier.md
│       ├── outreach-creator.md
│       └── outreach-verifier.md
│
├── APPLICATIONS/                  # Generated applications
│   └── [Company]_[Role]/
│       ├── JD.md
│       ├── RESUME.md
│       ├── COVERLETTER.md
│       ├── OUTREACH.md
│       └── [Company]_[Role]/
│           ├── Resume.docx
│           └── Coverletter.docx
│
├── PLAYBOOK/
│   ├── MASTER_TEMPLATE.md         # Resume format reference
│   ├── MASTER_RESUME.md           # Example bullets
│   ├── RESUME_FRAMEWORK.md        # Resume creation rules
│   ├── COVERLETTER_FRAMEWORK.md   # Cover letter templates
│   ├── OUTREACH_FRAMEWORK.md      # Outreach strategy guide
│   └── resume_generator.py        # DOCX conversion script
│
├── YOUR_PROFILE/
│   ├── USER_PROFILE.md            # Your professional profile
│   ├── USER_BULLETS.md            # Your bullet library
│   └── examples/
│       ├── EXAMPLE_USER_PROFILE.md
│       ├── EXAMPLE_USER_BULLETS.md
│       └── EXAMPLE_JD.md
│
├── CLAUDE.md                      # This file (required)
└── README.md                      # Setup guide
```

---

## Dependencies

- Python 3.x
- python-docx (`pip install python-docx`)
- Claude Code CLI

---

## Verification Commands

Character count verification:
```bash
echo "Your bullet text here" | wc -c
```

Word count verification:
```bash
echo "Your text here" | wc -w
```

---

## Troubleshooting

**"USER_PROFILE.md not found"**
→ Fill out `YOUR_PROFILE/USER_PROFILE.md` with your information (see examples folder)

**"No bullets found in USER_BULLETS.md"**
→ Fill out `YOUR_PROFILE/USER_BULLETS.md` with 40-60 accomplishment bullets

**"Bullet character count out of range"**
→ Adjust each bullet to exactly 240-260 characters: `echo "bullet text" | wc -c`

**"Summary character count out of range"**
→ Adjust summary to 360-380 characters: `echo "summary" | wc -c`

**"DOCX conversion failed"**
→ Run `pip install python-docx` and try again

**"Resume verification failed"**
→ Check RESUME.md meets all requirements:
  - 4 sections (Summary, Experience, Skills, Education)
  - 13 bullets (3-3-3-2-2 distribution)
  - All bullets 240-260 chars
  - No Certifications section
