---
name: ats-resume-maker
description: "Create ATS-optimized resumes with selective bold highlighting, two-page maximum, and export to DOCX/PDF. Use when: (1) Creating resumes from scratch or provided data, (2) Converting resume content to ATS-friendly format, (3) Generating DOCX/PDF resume files, (4) Validating resumes for ATS compliance"
---

# ATS Resume Maker

Create professional, ATS-optimized resumes. Maximum 2 pages.

## Workflow

### Step 1: Gather Data

If no resume data provided, collect from user:
- Contact: name, email, phone, LinkedIn, portfolio (optional)
- Summary: 2-3 sentence professional summary
- Skills: categorized technical skills
- Experience: company, title, dates, location, 3-5 bullets each
- Education: degree, institution, date, GPA (if >3.5)
- Optional: certifications, projects

### Step 2: Create Markdown

Follow the template in `references/resume_template.md`. Key structure:

```markdown
# FULL NAME
email | phone | linkedin | portfolio

## PROFESSIONAL SUMMARY
## TECHNICAL SKILLS
## PROFESSIONAL EXPERIENCE
## EDUCATION
## CERTIFICATIONS
```

### Step 3: Apply ATS Rules

See `references/ats_guidelines.md` for complete rules. Key points:

**Page limit:** Maximum 2 pages. Reduce bullets for older roles if needed.

**Selective bolding:** Only bold highlight-worthy items:
- ✅ Quantified metrics: **60% improvement**, **$2M revenue**
- ✅ Impressive numbers: **50+ sources**, **80% coverage**
- ❌ Don't bold: technology names, action verbs, company names

**No em dashes:** Use hyphens, commas, or pipes instead.

### Step 4: Validate

```bash
python scripts/validate_resume.py resume.md
```

Checks: page estimate, em dashes, word repetition, excessive bolding, structure.

### Step 5: Generate DOCX

```bash
npm install docx  # First time only
node scripts/generate_docx.js resume.md output.docx
```

### Step 6: Generate PDF

```bash
soffice --headless --convert-to pdf output.docx
```

## Output

Save all files to `/mnt/user-data/outputs/`:
- `{Name}_Resume.md`
- `{Name}_Resume.docx`
- `{Name}_Resume.pdf`
