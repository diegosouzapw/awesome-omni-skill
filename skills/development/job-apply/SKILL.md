---
name: job-apply
description: Fill out and submit job applications on supported platforms
user-invocable: true
---

# Job Application Skill

Fill out and submit job applications across LinkedIn, Indeed, Glassdoor, and ZipRecruiter.

## General Approach

For every application:
1. Navigate to the job posting URL
2. Read the full job description
3. Verify the job is still a good match (check requirements vs USER.md)
4. Click the apply button
5. Fill in all form fields using data from USER.md
6. Upload resume from `/home/ubuntu/job-apply/data/resume.pdf`
7. Generate a cover letter if required (follow SOUL.md rules)
8. Review the application before submitting
9. Submit
10. Take a screenshot of the confirmation page
11. Call `log_application` with the result

## Platform-Specific Instructions

### LinkedIn Easy Apply

1. On the job page, click the **"Easy Apply"** button
2. LinkedIn shows a multi-step modal form:
   - **Step 1 - Contact info:** Verify name, email, phone (pre-filled from LinkedIn profile). Correct if needed.
   - **Step 2 - Resume:** Click "Upload resume" and select `/home/ubuntu/job-apply/data/resume.pdf`. Remove any previously attached resume if it's not the current one.
   - **Step 3 - Additional questions:** These vary by job. Common fields:
     - Years of experience: Use value from USER.md
     - Work authorization: Use value from USER.md
     - Sponsorship required: Use value from USER.md
     - Willing to relocate: Based on USER.md location preferences
     - Salary expectations: Use USER.md salary range (pick the midpoint if a single number is required)
     - Start date: "Immediately" or "2 weeks notice" (use USER.md notes)
     - Cover letter: Generate one if the field is present
   - **Step 4 - Review:** Verify all fields, then click **"Submit application"**
3. Wait for confirmation message ("Your application was sent")
4. Take screenshot
5. Log with status "applied", platform "linkedin"

### LinkedIn Standard Apply (External)

1. Click **"Apply"** (non-Easy Apply) — this opens the company's external careers page
2. Wait for the external page to load fully
3. Look for the application form. Common patterns:
   - Greenhouse: `/jobs/[id]` forms
   - Lever: `jobs.lever.co/[company]/[id]`
   - Workday: `[company].wd5.myworkdayjobs.com`
   - Custom ATS or company website
4. Fill in standard fields:
   - First name, Last name
   - Email, Phone
   - Resume upload (find file input, upload PDF)
   - LinkedIn URL
   - Portfolio/GitHub URL
   - Cover letter (generate if required)
   - Work authorization questions
   - Demographic questions (skip or select "Decline to answer" / "Prefer not to say")
5. Submit and screenshot
6. Log result

### Indeed

1. On the job page, click **"Apply now"** or **"Apply on company site"**
2. If "Apply now" (Indeed's application):
   - Indeed shows a multi-step form:
     - Contact info: Verify/fill name, email, phone
     - Resume: Upload PDF or confirm Indeed resume
     - Questions: Answer job-specific questions using USER.md
     - Review & Submit
3. If "Apply on company site":
   - Follow the external site flow (similar to LinkedIn Standard Apply)
4. Screenshot confirmation
5. Log with platform "indeed"

### Glassdoor

1. On the job page, click **"Apply"** or **"Easy Apply"**
2. If Glassdoor Easy Apply:
   - Fill contact info
   - Upload resume
   - Answer additional questions
   - Submit
3. If external redirect:
   - Follow external site flow
4. Screenshot and log with platform "glassdoor"

### ZipRecruiter

1. On the job page, click **"Apply"** or look for **"1-Click Apply"**
2. If 1-Click Apply:
   - Confirm your ZipRecruiter profile details
   - Click submit
3. If standard apply:
   - Fill in the form fields
   - Upload resume
   - Submit
4. Screenshot and log with platform "ziprecruiter"

## Form Field Mapping

Map these common form fields to USER.md values:

| Form Field | USER.md Source |
|---|---|
| First Name / Last Name | Name (split on space) |
| Email | Email |
| Phone | Phone |
| City / Location | Location |
| LinkedIn URL | LinkedIn |
| GitHub / Portfolio | GitHub, Portfolio |
| Years of Experience | Years of Experience |
| Current Job Title | Derive from target roles |
| Desired Salary | Salary Range (midpoint or range) |
| Work Authorization | Work Authorization > Status |
| Sponsorship | Work Authorization > Sponsorship Required |
| Skills | Skills section |
| Education | Education section |

## Handling Dropdowns

- For dropdown menus, click to open, then look for the matching option
- If no exact match, pick the closest option
- For experience level dropdowns: map years to typical labels:
  - 0-2 years → "Entry level" or "Junior"
  - 3-5 years → "Mid-level" or "Associate"
  - 6-9 years → "Senior"
  - 10+ years → "Lead" or "Principal"

## Resume Upload

- The resume file is at `/home/ubuntu/job-apply/data/resume.pdf`
- Look for file input elements (`<input type="file">`)
- If there's a drag-and-drop zone, try clicking it to trigger the file dialog
- If upload fails, try finding alternative upload methods on the page

## Cover Letter Generation

When a cover letter is required:
1. Read the full job description
2. Generate a tailored cover letter following SOUL.md guidelines
3. Paste it into the cover letter field
4. Save the generated text in the `log_application` call (cover_letter field)

## Post-Submission

After each application:
1. Take a screenshot: save to `/home/ubuntu/screenshots/[platform]-[company]-[timestamp].png`
2. Call `log_application` with:
   - `platform`: linkedin | indeed | glassdoor | ziprecruiter
   - `company`: Company name
   - `title`: Job title
   - `url`: Job posting URL
   - `status`: applied | failed | skipped
   - `notes`: Any relevant notes (why it failed/was skipped)
   - `screenshotPath`: Path to the screenshot
3. Wait 30+ seconds before the next application (SOUL.md rate limiting)

## Error Recovery

- **Page won't load:** Wait 30 seconds, try once more, then skip
- **Form validation error:** Read the error message, try to fix, retry once
- **CAPTCHA appears:** Invoke the `captcha-solve` skill
- **"Already applied" message:** Log as "skipped" with note "already applied on platform"
- **Application limit reached:** Stop and notify user via WhatsApp
