---
name: resume-craft
description: Craft tailored, honest, one-page resumes from job descriptions. Use when the user wants to create, tailor, or improve a resume for a specific job posting. Handles job description analysis, skill gap identification, resume writing, keyword optimization, and PDF generation. Triggers on resume, CV, job application, tailor my resume, craft resume, apply for this job.
---

# Resume Crafting Skill

Analyze job descriptions and generate tailored, honest, one-page resumes. Produces Markdown source and professional PDF output via bundled script.

## How to Use

**Basic: provide a job description and your background**
```
Tailor my resume for this role:

[paste job description or URL]

My background:
- 5 years as software engineer at TechCorp
- Led team of 4 on cloud migration project
- Python, AWS, Terraform
- B.S. Computer Science
```

**With existing resume**
```
Tailor my resume for this position:

Job: [paste JD]
My resume: [paste or attach current resume]
```

**Career transition**
```
I'm moving from DevOps to product management. Tailor my resume for:
[paste JD]

Transferable experience:
- Led cross-functional infrastructure projects
- Managed stakeholder relationships as TAM
- Built internal tools adopted by 100+ users
```

## Core Principles

1. **Ask questions, don't make things up.** Unsure about a claim? Ask the user. Never fabricate experience, metrics, or titles.
2. **Accuracy over seniority.** A truthful resume beats an inflated one every time.
3. **One page, no exceptions.** Trim bullets, consolidate skills, cut filler.
4. **Match the audience.** A regional bank reads differently than a FAANG. Adjust tone, terminology, and emphasis to fit.
5. **Keywords matter, but naturally.** Incorporate JD language without keyword-stuffing.

## Workflow

### 1. Gather Materials

Check for the user's source resume or experience document. If none exists, ask for:
- Current resume or experience summary
- Target role URL or job description text
- Specific preferences or constraints

**Do not proceed without understanding the user's real background.** If information is vague, ask clarifying questions before writing.

### 2. Analyze the Job Description

- Separate must-have vs nice-to-have requirements
- Note specific technologies, tools, certifications mentioned
- Identify skill gaps between user's experience and JD
- Flag red flags (vague JD, "wear many hats", excessive "fast-paced") and green flags (salary disclosed, clear tech stack, team structure)
- Determine the audience: startup vs enterprise, technical vs business, FAANG vs regional

### 3. Craft the Resume

#### Header
- `# FULL NAME` (18pt bold, centered)
- Contact info on one line, pipe-separated: `City, State | email | phone | linkedin`
- **Never put the target role as a subtitle.** That implies you currently hold that title. Use no subtitle or the candidate's actual title/tagline.

#### Summary (2-3 sentences max)
- Lead with years of experience + core domains + industries
- Keep claims realistic and grounded
- Avoid filler: "proven track record", "results-driven", "passionate", "dynamic"
- Match tone to audience -- "senior technical advisor" not "executive advisor" for non-FAANG

#### Skills (4-5 rows max)
- Lead with skills mentioned in the JD
- Remove skills not relevant to the role
- Format: `**Category:** Skill1, Skill2, Skill3`
- Do NOT use markdown tables -- the PDF parser can't handle them
- Include certifications inline when relevant: `AWS (Solutions Architect Pro)`

#### Experience (reverse chronological)
- **Always list in strict reverse chronological order.** Most recent first. Double-check dates before finalizing.
- Reorder bullets within each role to lead with most JD-relevant
- Rewrite bullets using JD keywords naturally
- Quantify impact: users, %, time saved, $ amounts
- Say "internal users" for internal tools (not "users" which implies external)
- Format: `[Action Verb] + [What] + [How/Result] + [Impact/Metric]`
- Trim older roles to 2 bullets max
- Remove bullets that don't support the target role

#### Certifications & Education
- List relevant certifications by name, not count ("Solutions Architect Pro" not "12 certifications")
- For expired certs: omit dates or say "earned 2020"
- Include degree, school, and year

#### Community / Advocacy (optional)
- One line max. Don't mix unrelated info here.

### 4. One Page Check

If the resume spills to a second page:
1. Trim least relevant bullets first
2. Consolidate skills rows
3. Shorten section headers
4. Remove filler words
5. Reduce spacing only as last resort

### 5. Post-Generation Review

After presenting the tailored resume, provide:

**Strengths:** What makes this candidate competitive for this specific role
**Gaps:** Requirements not fully met and how to address them (courses, projects, reframing)
**Recommendations:** Cover letter hooks, interview talking points, or skills to highlight

### 6. Generate PDF

Convert Markdown to professional PDF using the bundled script:

```bash
pip install reportlab  # one-time setup
python scripts/generate-resume-pdf.py resume.md resume.pdf
```

## Markdown Format for PDF Compatibility

The PDF script parses a specific markdown structure. Follow this exactly:

```markdown
# FULL NAME

City, State | email@example.com | 555-123-4567 | linkedin.com/in/handle

---

## PROFESSIONAL SUMMARY

Two to three sentences. Grounded and specific.

## SKILLS

**Category One:** Skill A, Skill B, Skill C
**Category Two:** Skill D, Skill E, Skill F

## PROFESSIONAL EXPERIENCE

### Company Name | City, State
**Job Title** | Start Date -- End Date

- Quantified achievement bullet
- Another achievement bullet

## CERTIFICATIONS & EDUCATION

**Certifications:** Cert A, Cert B
**Education:** B.S. Field of Study, University Name, Year
```

**Critical rules:**
- `**bold**` for skill categories and role titles, never markdown tables
- Contact info uses ` | ` pipe separators on one line
- `---` only once (after contact info) as header divider
- `## ` for section headers, `### ` for company names, `- ` for bullets

## Special Considerations

**Career changers:** Lead with transferable skills. Reframe past experience using the target role's language. Address the transition in the summary.

**Technical roles:** Include a prominent skills section. List languages, frameworks, tools. Link to GitHub/portfolio if relevant.

**Senior / executive roles:** Focus on strategic impact, team building, revenue. Keep technical details lighter. Can extend to 2 pages if 15+ years.

**Recent graduates:** Lead with education. Include relevant coursework, projects, internships. Emphasize leadership in student orgs.

**Non-traditional backgrounds (no degree, career gap, self-taught):** Lead with experience and certifications. Don't hide gaps -- frame them positively. Certifications and projects speak louder than degrees.

## PDF Spacing Reference

| Element | spaceBefore | spaceAfter |
|---------|-------------|------------|
| Name (H1) | -- | 8pt |
| Contact line | -- | 8pt |
| Section header (H2) | 8pt | 6pt |
| Company header (H3) | 4pt | 1pt |
| Role title | -- | 3pt |
| Body text | -- | 3pt |
| Bullet points | -- | 2pt |
| Skill rows | -- | 2pt |

## Common Mistakes

| Don't | Do |
|-------|-----|
| Put target role as subtitle | No subtitle or actual current title |
| Use markdown tables for skills | `**Category:** items` format |
| "12 certifications" | List by name: Solutions Architect Pro, DevOps Pro |
| "proven track record" | Cut it |
| Claim titles you don't hold | Describe what you did, not what you want to be |
| "C-suite stakeholder management" | "Executive stakeholder engagement" |
| List experience out of order | Reverse chronological -- double-check dates |
| Overclaim AI/emerging tech | "Applied AI", "GenAI prototyping", "AI-driven automation" |
| VP-level tone for all audiences | Match tone to company size and culture |
| Fabricate metrics | Ask the user for real numbers or omit |
| Include every skill you have | Only skills relevant to this JD |
| Generic summary for all jobs | Rewrite summary for each application |

## Tips for Best Results

- **Be specific:** Provide the complete job description, not just a title
- **Share real metrics:** Numbers, percentages, team sizes, dollar amounts
- **Mention constraints:** Page limits, format preferences, sections to include/exclude
- **Iterate:** Ask for revisions, alternative emphasis, or different tone
- **Multiple roles:** Generate separate tailored versions for each application
- **Review before sending:** Always verify accuracy -- the skill optimizes, you verify
