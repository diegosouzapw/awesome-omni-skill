---
name: add-member
description: Interactive guide for adding new lab members to the website. Gathers information via web research, helps write appropriate bios, optimizes images, and generates properly formatted member files. Use when adding new team members, updating member info, or marking members as alumni.
version: 1.0.0
allowed-tools: Read, Write, Edit, Bash, WebSearch, AskUserQuestion, Glob
---

# Add Lab Member

This skill guides you through adding a new lab member to the website with proper formatting, bio writing, and image handling.

## When to Use This Skill

Use this skill when:
- "Add a new lab member"
- "Create a profile for [name]"
- "Add [name] to the team page"
- "Update [name]'s role"
- "Mark [name] as alumni"
- Any request involving adding or updating team member profiles

## Interactive Workflow

### Step 1: Gather Basic Information

Ask the user for essential details:

**Required**:
- Full name
- Role (pi, postdoc, staff, phd, ms, ra, programmer, undergrad, highschool)

**Optional** (we'll research if not provided):
- Email address
- GitHub username
- LinkedIn profile
- Personal website
- Current institution/program (if student)

**Use AskUserQuestion** to collect this information if not provided in the initial request.

### Step 2: Web Research

Conduct web research to gather background information:

1. **Search LinkedIn** for education and work history
   - Query: "[Full Name] LinkedIn [institution if known]"
   - Extract: degrees, institutions, prior positions

2. **Search GitHub** for username and projects
   - Query: "[Full Name] GitHub [email domain if known]"
   - Extract: username, notable repositories

3. **Search institutional pages** for bio information
   - Query: "[Full Name] [institution] neuroscience/biology/computer science"
   - Extract: research interests, current work

4. **Search publications** if applicable (PhD/postdoc)
   - Query: "[Full Name] [institution] publications"
   - Extract: research areas

**Present findings to user**:
- Show what you found
- Ask user to confirm accuracy
- Ask for clarification on anything unclear
- Request any missing critical information

### Step 3: Draft Bio

Using the gathered information and role-appropriate template:

1. **Determine bio length based on role**:
   - PI: 250-300 words (detailed)
   - PhD/Postdoc/Staff: 100-150 words
   - MS/RA/Programmer: 80-120 words
   - Undergrad: 80-100 words
   - Alumni: 50-250 words (based on tenure and achievements)

2. **Follow bio template** from templates/ directory:
   - Use third-person voice
   - Professional, academic tone
   - Include: education, join date, research focus/current work
   - Mention co-advisors if applicable
   - Highlight prior experience if relevant
   - Note awards/fellowships if notable

3. **Present draft to user**:
   - Show the drafted bio
   - Ask if they want to make any edits
   - Refine based on feedback

### Step 4: Handle Profile Image

1. **Ask user for image**:
   - "Do you have a profile photo for [name]?"
   - If yes: "Please provide the path to the image file"
   - If no: "We can use a placeholder for now and add it later"

2. **If image provided, validate and optimize**:
   - Check file size and dimensions using `file` or `sips`
   - Target specs: 100-250 KB, reasonably square aspect ratio
   - If too large (>500 KB) or wrong aspect ratio:
     - Offer to resize/optimize using scripts/optimize-image.sh
     - Use `sips` (macOS) or `convert` (ImageMagick) if available

3. **Generate image filename**:
   - Format: `firstname-lastname.jpg`
   - Ensure no spaces or special characters
   - Copy to `images/` directory with correct name

### Step 5: Generate Member File

1. **Create filename**: `_members/firstname-lastname.md`

2. **Build frontmatter**:
   ```yaml
   ---
   name: [Full Name]
   image: images/[firstname-lastname].jpg
   role: [role]
   description: [Role display text from roles.yaml]
   links:
     email: [email if provided]
     github: [username if found/provided]
     linkedin: [profile if found/provided]
     home-page: [url if provided]
   ---
   ```

   - Only include links that are available
   - Remove empty link fields

3. **Add bio text** (from Step 3)

4. **Write file** to `_members/firstname-lastname.md`

### Step 6: Handle Alumni (if applicable)

If the role is `alumni` OR user indicates this person is leaving:

1. **Ask for alumni details**:
   - Year they left/are leaving
   - Their role while in the lab (if different from current)
   - "Next" position (where they're going)

2. **Update team/index.md**:
   - Read current alumni section
   - Add entry in chronological order (most recent first):
     ```markdown
     - YEAR: [**Name**](/members/firstname-lastname.html) (Role Description). **Next:** [Next position]
     ```
   - Maintain proper formatting

### Step 7: Validate

Run validation checks:

- [ ] Member file exists at `_members/firstname-lastname.md`
- [ ] YAML frontmatter is valid
- [ ] Role is valid (check against roles.yaml)
- [ ] Image file exists (or noted as placeholder)
- [ ] Bio is appropriate length for role
- [ ] Bio uses third-person voice
- [ ] Links are properly formatted
- [ ] If alumni: team/index.md is updated

### Step 8: Present Summary

Show the user what was created:

```
✓ Created member profile for [Name]
✓ File: _members/firstname-lastname.md
✓ Role: [role display text]
✓ Image: images/firstname-lastname.jpg [or "Placeholder - add later"]
✓ Links: [list of included links]
[✓ Updated alumni list] [if applicable]

Bio preview:
[First 100 characters of bio]...

Would you like me to show you the full file content?
```

### Step 9: Optional Commit

Ask user if they want to commit the changes:
- If yes: Create git commit with appropriate message
- If no: Leave as unstaged changes for manual review

## Bio Writing Guidelines

### Voice and Tone
- **Always third person**: "Jane is a PhD student..." (not "I am...")
- **Professional and academic**: Formal but not stuffy
- **Factual and achievement-oriented**: Focus on credentials and work
- **Present tense for current work**: "working on", "developing", "researching"
- **Past tense for background**: "received", "worked", "joined"

### Essential Elements

**All members should include**:
1. Current role and institution
2. When they joined (month and year)
3. Education background (degree, major, institution)
4. Current research/work focus

**Additional elements by role**:

**PI**:
- PhD institution and advisors (with hyperlinks)
- Research philosophy and approach
- Notable publications or media coverage
- Prior positions
- Major awards and honors

**PhD/Postdoc**:
- Undergraduate and/or Master's institution
- Specific research interests
- Co-advisors if applicable
- Previous research experience if notable

**Staff/Programmer**:
- Prior work experience (industry or academic)
- Technical expertise and focus areas
- Projects they're working on
- Joint appointments if applicable

**Students (MS/Undergrad)**:
- Major and year
- Research interests
- Prior internships or research if applicable
- Awards/fellowships if notable

**Alumni**:
- All of above based on their role
- Mentors (especially for short-term interns)
- Achievements during time in lab
- Keep concise for summer interns (50-80 words)

### What to Avoid

- First person narrative (except possibly PI intro)
- Excessive detail about projects (keep to 1-2 sentences)
- Future tense or speculation
- Casual language or slang
- Personal information unrelated to research
- Redundancy with frontmatter (name, role already shown)

## Image Handling Details

### Preferred Specifications
- **Format**: JPG (preferred) or PNG
- **Size**: 100-250 KB for web optimization
- **Aspect ratio**: Square or nearly square (1:1 or 4:5)
- **Minimum dimensions**: 300x300 pixels
- **Recommended**: 400x400 to 600x600 pixels
- **Color**: RGB color space

### Optimization Process

If image needs optimization:

```bash
# Use the bundled script
bash .claude/skills/add-member/scripts/optimize-image.sh input.jpg output.jpg 500
```

Or manually with sips (macOS):
```bash
# Resize to 500px max dimension, maintaining aspect ratio
sips -Z 500 input.jpg --out output.jpg

# Convert PNG to JPG
sips -s format jpeg input.png --out output.jpg
```

Or with ImageMagick:
```bash
# Resize and optimize
convert input.jpg -resize 500x500^ -quality 85 output.jpg
```

### Filename Convention
- Use kebab-case: `firstname-lastname.jpg`
- No spaces, special characters, or uppercase
- Match the member file slug
- Examples: `jane-doe.jpg`, `john-smith.jpg`

## Role-Specific Notes

### Adding Alumni

When adding someone who has already left:

1. Set `role: alumni` in frontmatter
2. Include their actual role in bio text
3. Update `team/index.md` alumni section with:
   - Year they left
   - Role description in parentheses
   - "Next:" annotation if known

### Handling Co-Advisors

When member has co-advisor:

1. Mention in bio: "co-advised by [Name]"
2. Include hyperlink if possible: `[Name](url)`
3. Note in summary when presenting to user

### Joint Appointments

When member has joint affiliation:

1. Mention in bio: "jointly with the [Lab Name]"
2. Include hyperlink to other lab if available
3. Note collaborating PIs clearly

### Students Transitioning Roles

When student moves from undergrad → MS → PhD:

1. Create new member file? **No** - keep same file
2. Update `role:` field in frontmatter
3. Update bio to reflect new status
4. Update `description:` field
5. Consider adding a note about the transition in bio

### Summer Interns

Short-term members (2-3 months):

1. **While active**: Add with role `undergrad` or `highschool`
2. **When leaving**: Change to `role: alumni`
3. **Bio**: Can be brief (50-80 words) but highlight:
   - Institution and program
   - What they worked on
   - Who mentored them
   - Any notable achievements

## Error Handling

**If web search finds nothing**:
- Ask user to provide information directly
- Offer to draft minimal bio and let them expand

**If image is wrong format/size**:
- Offer to optimize using provided tools
- If tools not available, provide manual instructions
- Worst case: use placeholder and note in summary

**If unsure about role**:
- Present role options from roles.yaml
- Use AskUserQuestion to clarify

**If duplicate member exists**:
- Check if it's an update vs new member
- Ask user if they want to update existing or create new

## Validation Checklist

Before finalizing, verify:

- [ ] Full name matches across filename, frontmatter, and bio
- [ ] Role is valid (one of: pi, postdoc, staff, phd, ms, ra, programmer, undergrad, highschool, alumni)
- [ ] Image filename follows convention (kebab-case, .jpg)
- [ ] Bio is third person throughout
- [ ] Bio length appropriate for role (see guidelines)
- [ ] Join date mentioned in bio (month and year)
- [ ] Education background included
- [ ] All links are properly formatted
- [ ] YAML frontmatter is valid (test with parser)
- [ ] If alumni: team/index.md updated correctly
- [ ] No trailing whitespace or formatting issues

## See Also

- [reference.md](reference.md) - Detailed documentation on member file format and patterns
- [examples.md](examples.md) - Example workflows for different scenarios
- [templates/](templates/) - Bio templates for each role type
