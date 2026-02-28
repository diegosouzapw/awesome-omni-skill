---
name: analyze-job-post
description: Analyzes a job offer for fit with the Candidate's profile and recommends an application strategy. Use before considering a new job offer.
---

# Analyze Job Post

<workflow>

## Step 1: Data Extraction

1. Read the job offer from the user request.
2. Extract: Role, Company, Location, Salary, Requirements (Must-have/Nice-to-have).

## Step 2: Competency Fit Analysis

1. Compare requirements with `Candidate-Profile.md`.
2. Identify:
   - **Strong Matches:** Direct experience.
   - **Transferable Skills:** Adaptable skills.
   - **Gaps:** Missing qualifications.
   - **Differentiators:** Unique strengths.

## Step 3: Context & Positioning

1. Research company context (Size, Sector, Culture).
2. Determine Organization Type (Tech, Corp, NGO, Public).
3. Select optimal Positioning Strategy from Profile (e.g., Tech Expert vs. Business Pro).

## Step 4: Recommendation

1. Assess success probability (High/Medium/Low).
2. Create an analysis using the template in `references/templates.md`.
3. Provide concrete next steps (Apply/Skip/Research).

</workflow>
