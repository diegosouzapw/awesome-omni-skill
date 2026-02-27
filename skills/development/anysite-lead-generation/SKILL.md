---
name: anysite-lead-generation
description: Lead generation and prospecting using anysite MCP server for LinkedIn prospect discovery, email finding, company research, and contact enrichment. Extract contacts from websites, find decision-makers at target companies, and build qualified prospect lists for sales, recruiting, and business development. Supports LinkedIn (primary), web scraping for contact extraction, and Instagram business discovery. Use when users need to build prospect lists, find decision-makers, extract contact information, research potential customers, or enrich existing leads with additional data.
---

# anysite Lead Generation

Professional lead generation and prospecting using the anysite MCP server. Find prospects on LinkedIn, discover verified emails, extract contacts from websites, and build comprehensive lead lists for sales, recruiting, and business development.

## Overview

The anysite Lead Generation skill helps you:
- **Find qualified prospects** on LinkedIn using advanced search filters
- **Enrich profiles** with work history, education, and skills
- **Discover email addresses** through LinkedIn email finding
- **Extract contact information** from company websites
- **Research companies** to identify target accounts
- **Build prospect lists** formatted for CRM import

This skill provides comprehensive lead generation capabilities with the added benefit of zero configuration and immediate execution through anysite MCP.

## Supported Platforms

- ✅ **LinkedIn** (Primary): People search, profile enrichment, email discovery, company research, employee listings
- ✅ **Web Scraping**: Contact extraction from websites, sitemap parsing, general web data
- ✅ **Instagram**: Business account discovery and profile analysis (supplementary)
- ✅ **Y Combinator**: Startup company and founder research (supplementary)

## Quick Start

### Step 1: Identify Your Lead Source

Choose the appropriate data source based on your prospecting goal:

| Goal | Primary Tool | Use Case |
|------|-------------|----------|
| Find prospects by title/company | `search_linkedin_users` | B2B prospecting, targeted outreach |
| Enrich existing leads | `get_linkedin_profile` | Add work history, education, skills |
| Find verified emails | `find_linkedin_user_email` | Email outreach campaigns |
| Extract website contacts | `parse_webpage` | Get emails/phones from contact pages |
| Research target companies | `search_linkedin_companies` | Account-based marketing (ABM) |
| Find company employees | `search_linkedin_users` + company filter | Multi-threading into accounts |

### Step 2: Execute Data Collection

Use anysite MCP tools directly to gather lead data:

**Example: Find Sales VPs in San Francisco**
```
Tool: mcp__anysite__search_linkedin_users
Parameters:
- title: "VP Sales"
- location: "San Francisco Bay Area"
- count: 25
```

**Example: Enrich a LinkedIn Profile**
```
Tool: mcp__anysite__get_linkedin_profile
Parameters:
- user: "linkedin.com/in/johndoe" or "johndoe"
- with_experience: true
- with_education: true
- with_skills: true
```

**Example: Find Email Address**
```
Tool: mcp__anysite__find_linkedin_user_email
Parameters:
- email: "john@company.com" (for reverse lookup)
- count: 5
```

**Example: Extract Contacts from Website**
```
Tool: mcp__anysite__parse_webpage
Parameters:
- url: "https://company.com/contact"
- extract_contacts: true
- strip_all_tags: true
```

### Step 3: Process and Analyze Results

Review the returned data for:
- **Profile completeness**: Work history, education, skills presence
- **Contact quality**: Email deliverability, phone format
- **Relevance scoring**: Title match, company fit, location alignment
- **Enrichment opportunities**: Missing data that can be filled

### Step 4: Format Output

Choose your preferred output format:

**Chat Summary (Default)**
- Displays top prospects with key details
- Includes actionable next steps
- Shows data quality metrics

**CSV Export**
- Full prospect list with all fields
- Ready for CRM import (Salesforce, HubSpot, etc.)
- Includes: Name, Title, Company, Location, Email, LinkedIn URL, Work History

**JSON Export**
- Structured data for custom integration
- Complete API response with metadata
- Ideal for automation and data pipelines

## Common Workflows

### Workflow 1: LinkedIn B2B Prospecting

**Scenario**: Find 50 qualified prospects at SaaS companies in specific roles

**Steps**:

1. **Search for prospects**
```
Tool: mcp__anysite__search_linkedin_users
Parameters:
- keywords: "SaaS software cloud"
- title: "Head of Marketing, VP Marketing, CMO"
- location: "United States"
- company_keywords: "software"
- count: 50
```

2. **Enrich top prospects** (for first 10-20 results)
```
For each prospect from step 1:
Tool: mcp__anysite__get_linkedin_profile
Parameters:
- user: <prospect_username>
- with_experience: true
- with_education: true
- with_skills: false (unless skills matter for qualification)
```

3. **Find email addresses**
```
For each qualified prospect:
Tool: mcp__anysite__find_linkedin_user_email
Then try:
Tool: mcp__anysite__get_linkedin_user_email_db
```

4. **Export to CRM**
- Format as CSV with fields: Full Name, Title, Company, Location, Email, LinkedIn URL
- Import to your CRM system
- Set up outreach sequences

**Expected Output**:
- 50 prospects with LinkedIn profiles
- 10-20 enriched profiles with complete work history
- 5-15 verified email addresses
- CSV file ready for CRM import

### Workflow 2: Account-Based Marketing (ABM)

**Scenario**: Find decision-makers at specific target companies

**Steps**:

1. **Research target companies**
```
Tool: mcp__anysite__search_linkedin_companies
Parameters:
- keywords: <your ICP description>
- industry: <target industry>
- employee_count: ["51-200", "201-500"]
- location: <target location>
- count: 20
```

2. **Get company details**
```
For each target company:
Tool: mcp__anysite__get_linkedin_company
Parameters:
- company: <company_identifier or URL>
```

3. **Find employees at target companies**
```
For each target company:
Tool: mcp__anysite__search_linkedin_users
Parameters:
- company_keywords: "<Company Name>"
- title: "VP, Director, Head of, Chief"
- count: 10
```

4. **Enrich decision-makers**
```
For each decision-maker:
Tool: mcp__anysite__get_linkedin_profile
Tool: mcp__anysite__find_linkedin_user_email
```

5. **Create ABM campaign**
- Group prospects by company
- Identify multi-threading opportunities
- Build company-specific messaging

**Expected Output**:
- 20 target companies with full profiles
- 50-100 decision-makers across all companies
- 20-40 verified email addresses
- Account map showing org structure

### Workflow 3: Website Contact Extraction

**Scenario**: Extract contact information from a list of company websites

**Steps**:

1. **Get company websites**
```
From LinkedIn company search or existing list
Tool: mcp__anysite__get_linkedin_company
Extract: company website URLs
```

2. **Parse contact pages**
```
For each website:
Tool: mcp__anysite__parse_webpage
Parameters:
- url: "https://company.com/contact"
- extract_contacts: true
- strip_all_tags: true

Alternative pages to try:
- /contact
- /about/contact
- /about/team
- /about
```

3. **Parse team pages** (if available)
```
Tool: mcp__anysite__parse_webpage
Parameters:
- url: "https://company.com/team"
- extract_contacts: true
```

4. **Get sitemap** (for comprehensive coverage)
```
Tool: mcp__anysite__get_sitemap
Parameters:
- url: "https://company.com"
- count: 50

Identify pages likely to contain contacts:
- /contact, /team, /about, /leadership
```

5. **Deduplicate and validate**
- Remove duplicate emails
- Validate email formats
- Match emails to LinkedIn profiles if possible

**Expected Output**:
- Contact emails from 60-80% of websites
- Phone numbers where available
- Social media links
- Team member names and titles

### Workflow 4: Recruiter Candidate Sourcing

**Scenario**: Find qualified candidates for open positions

**Steps**:

1. **Define candidate profile**
```
Required skills, titles, experience level, location
```

2. **Search for candidates**
```
Tool: mcp__anysite__search_linkedin_users
Parameters:
- keywords: <technical skills, e.g., "Python React AWS">
- title: <relevant titles, e.g., "Software Engineer, Senior Engineer">
- location: <location or "Remote">
- count: 100
```

3. **Enrich candidate profiles**
```
For promising candidates:
Tool: mcp__anysite__get_linkedin_profile
Tool: mcp__anysite__get_linkedin_user_experience
Tool: mcp__anysite__get_linkedin_user_skills
Tool: mcp__anysite__get_linkedin_user_education
```

4. **Find contact information**
```
Tool: mcp__anysite__find_linkedin_user_email
Tool: mcp__anysite__get_linkedin_user_email_db
```

5. **Build candidate pipeline**
- Score candidates on skills match
- Prioritize by years of experience
- Create outreach sequence

**Expected Output**:
- 100 potential candidates
- 30-50 fully enriched profiles
- 20-30 email addresses
- Scored and prioritized candidate list

## MCP Tools Reference

### Primary Tools

#### LinkedIn People Search
**Tool**: `mcp__anysite__search_linkedin_users`

Search for LinkedIn users by various criteria.

**Parameters**:
- `keywords` (optional): General keywords for search
- `title` (optional): Job title keywords (e.g., "VP Sales", "Software Engineer")
- `company_keywords` (optional): Company name keywords
- `location` (optional): Location (city, state, country)
- `school_keywords` (optional): School/university keywords
- `first_name` (optional): First name
- `last_name` (optional): Last name
- `count` (default: 10): Number of results to return

**Returns**: List of user profiles with name, title, location, profile URL, and URN

**Use Cases**:
- Find prospects by title and location
- Discover employees at target companies
- Search for alumni from specific schools
- Build prospect lists for outreach

#### LinkedIn Profile Details
**Tool**: `mcp__anysite__get_linkedin_profile`

Get comprehensive profile information for a LinkedIn user.

**Parameters**:
- `user` (required): LinkedIn username or full profile URL
- `with_education` (default: true): Include education history
- `with_experience` (default: true): Include work experience
- `with_skills` (default: true): Include skills

**Returns**: Complete profile with work history, education, skills, certifications, and more

**Use Cases**:
- Enrich prospect data before outreach
- Qualify leads based on work history
- Research candidate backgrounds
- Understand prospect's career path

#### Email Finding
**Tool**: `mcp__anysite__find_linkedin_user_email`

Search for email addresses associated with LinkedIn profiles.

**Parameters**:
- `email` (required): Email address for reverse lookup
- `count` (default: 5): Number of results
- `request_timeout` (default: 300): Timeout in seconds

**Returns**: LinkedIn profiles associated with email addresses

**Use Cases**:
- Find verified emails for prospects
- Reverse lookup email to LinkedIn profile
- Enrich contact databases

#### Email Database Lookup
**Tool**: `mcp__anysite__get_linkedin_user_email_db`

Retrieve cached email address for a LinkedIn profile.

**Parameters**:
- `profile` (required): LinkedIn profile URL
- `request_timeout` (default: 300): Timeout in seconds

**Returns**: Email address if available in database

**Use Cases**:
- Quick email lookup for known profiles
- Batch email enrichment
- Contact data verification

#### Company Search
**Tool**: `mcp__anysite__search_linkedin_companies`

Search for LinkedIn companies by various criteria.

**Parameters**:
- `keywords` (optional): Company name or description keywords
- `location` (optional): Company location
- `industry` (optional): Industry type
- `employee_count` (optional): Array of employee count ranges (e.g., ["51-200", "201-500"])
- `count` (required): Number of results to return
- `request_timeout` (default: 300): Timeout in seconds

**Returns**: List of companies with name, industry, size, location, and URN

**Use Cases**:
- Identify target accounts for ABM
- Research competitors
- Build company prospect lists
- Market segmentation analysis

#### Company Details
**Tool**: `mcp__anysite__get_linkedin_company`

Get detailed information about a LinkedIn company.

**Parameters**:
- `company` (required): Company identifier or LinkedIn URL
- `request_timeout` (default: 300): Timeout in seconds

**Returns**: Company profile with description, industry, size, specialties, website

**Use Cases**:
- Research target accounts
- Understand company positioning
- Extract company websites for further research
- Qualify accounts before prospecting

#### Company Employee Stats
**Tool**: `mcp__anysite__get_linkedin_company_employee_stats`

Get employee statistics and insights for a company.

**Parameters**:
- `urn` (required): Company URN from company lookup
- `request_timeout` (default: 300): Timeout in seconds

**Returns**: Employee count, growth metrics, department distribution

**Use Cases**:
- Track hiring velocity (company growth)
- Identify growing departments
- Assess company size and scale
- Competitive intelligence on hiring

### Supporting Tools

#### Web Contact Extraction
**Tool**: `mcp__anysite__parse_webpage`

Extract content and contact information from web pages.

**Parameters**:
- `url` (required): Webpage URL
- `extract_contacts` (default: false): Extract emails, phones, social links
- `strip_all_tags` (default: true): Remove HTML tags
- `only_main_content` (default: true): Extract only main content area
- `same_origin_links` (default: false): Include only same-domain links
- `request_timeout` (default: 300): Timeout in seconds

**Returns**: Page content, contacts (emails, phones), links

**Use Cases**:
- Extract contacts from company websites
- Get email addresses from contact pages
- Find phone numbers and addresses
- Discover team member information

#### Sitemap Parsing
**Tool**: `mcp__anysite__get_sitemap`

Retrieve sitemap URLs from a website.

**Parameters**:
- `url` (required): Website URL
- `count` (default: 50): Number of URLs to return
- `request_timeout` (default: 300): Timeout in seconds

**Returns**: List of URLs from sitemap

**Use Cases**:
- Find all pages on a website
- Identify contact/team pages
- Comprehensive website crawling
- Discover hidden landing pages

#### LinkedIn User Experience
**Tool**: `mcp__anysite__get_linkedin_user_experience`

Get detailed work experience history for a LinkedIn user.

**Parameters**:
- `urn` (required): User URN from search or profile
- `count` (default: 10): Number of experience entries
- `request_timeout` (default: 300): Timeout in seconds

**Returns**: Detailed work history with dates, descriptions, companies

**Use Cases**:
- Deep work history analysis
- Career path research
- Qualification scoring
- Experience validation

#### LinkedIn User Skills
**Tool**: `mcp__anysite__get_linkedin_user_skills`

Get skills and endorsements for a LinkedIn user.

**Parameters**:
- `urn` (required): User URN
- `count` (default: 10): Number of skills to return
- `request_timeout` (default: 300): Timeout in seconds

**Returns**: Skills list with endorsement counts

**Use Cases**:
- Technical skills validation
- Expertise assessment
- Candidate qualification
- Skills-based matching

## Output Formats

### Chat Summary (Default)

The skill will provide:
- **Prospect count**: Total leads found
- **Key insights**: Notable patterns or high-value prospects
- **Quality metrics**: Percentage with emails, complete profiles, etc.
- **Top prospects**: 5-10 best matches with details
- **Next steps**: Recommended actions

Example:
```
Found 47 qualified prospects matching your criteria:

Top Prospects:
1. Jane Smith - VP Sales at TechCorp (San Francisco)
   - Email: jane.smith@techcorp.com
   - 10 years in SaaS sales leadership
   - Previously at Salesforce, Oracle

2. John Doe - Head of Sales at CloudCo (Remote)
   - Email: Not found (try web scraping)
   - 8 years experience
   - Strong enterprise background

Quality Metrics:
- 32% have verified emails (15/47)
- 91% have complete LinkedIn profiles (43/47)
- 68% in target location (32/47)

Next Steps:
1. Export to CSV for CRM import
2. Set up email outreach sequence
3. Prioritize prospects with emails
4. Research companies for remaining prospects
```

### CSV Export

Use CSV format for CRM import and spreadsheet analysis.

**How to Request**:
"Export the results as CSV"

**CSV Structure**:
```csv
Full Name,First Name,Last Name,Title,Company,Location,Email,Phone,LinkedIn URL,Profile URN,Years Experience,Education,Skills,Last Updated
```

**Use Cases**:
- Salesforce/HubSpot import
- Mail merge for outreach
- Spreadsheet analysis and filtering
- Team collaboration via Google Sheets

**CSV Benefits**:
- Universal format for all CRM systems
- Easy filtering and sorting
- Compatible with email tools
- Shareable with team members

### JSON Export

Use JSON format for programmatic access and custom integration.

**How to Request**:
"Export the results as JSON"

**JSON Structure**:
```json
{
  "prospects": [
    {
      "profile": {
        "firstName": "Jane",
        "lastName": "Smith",
        "headline": "VP Sales at TechCorp",
        "location": "San Francisco Bay Area",
        "profileUrl": "https://linkedin.com/in/janesmith",
        "urn": "urn:li:fsd_profile:ABC123"
      },
      "currentPosition": {
        "title": "VP Sales",
        "company": "TechCorp",
        "startDate": "2021-06"
      },
      "contact": {
        "email": "jane.smith@techcorp.com",
        "phone": null
      },
      "experience": [...],
      "education": [...],
      "skills": [...]
    }
  ],
  "metadata": {
    "total": 47,
    "withEmails": 15,
    "timestamp": "2026-01-29T01:00:00Z"
  }
}
```

**Use Cases**:
- Custom CRM integration
- Automated enrichment pipelines
- Data analysis with Python/R
- API integrations with other tools

## Advanced Features

### Multi-Platform Lead Enrichment

Combine LinkedIn data with other platforms for comprehensive lead profiles:

**Pattern**: LinkedIn → Company Website → Instagram (for B2C)

1. Find prospect on LinkedIn
2. Get company website from LinkedIn company profile
3. Extract additional contacts from website
4. Check Instagram for business account (B2C companies)
5. Analyze social presence and engagement

**Example**:
```
1. search_linkedin_users → Find "Emily Chen" at "FashionBrand"
2. get_linkedin_company("FashionBrand") → Get website URL
3. parse_webpage(website + "/contact") → Get phone, email
4. search_instagram_posts(query="FashionBrand") → Find official account
5. get_instagram_user("fashionbrand") → Verify business account, get follower count
```

### Boolean Search Patterns

Advanced LinkedIn search using keyword combinations:

**Title Combinations**:
- `"VP Sales" OR "Head of Sales" OR "Director Sales"` - Multiple title variations
- `"Software Engineer" AND "Python"` - Title + required skill
- `"Marketing" NOT "Intern"` - Exclude junior levels

**Company Patterns**:
- `"Google OR Meta OR Amazon"` - Multiple target companies
- `"SaaS" AND "B2B"` - Industry qualifiers
- `"Startup OR Early Stage"` - Company stage targeting

**Location Strategies**:
- `"San Francisco Bay Area"` - Metropolitan areas
- `"United States"` - Country-level
- `"Remote"` - Remote-only positions

### Lead Scoring Framework

Score prospects based on multiple criteria:

**Scoring Dimensions**:
1. **Profile Completeness** (0-20 points)
   - Has email: +10
   - Complete work history: +5
   - Complete education: +3
   - Skills listed: +2

2. **Title Match** (0-30 points)
   - Exact match: +30
   - Similar title: +20
   - Related title: +10
   - Wrong level: +0

3. **Company Fit** (0-25 points)
   - Ideal company size: +25
   - Acceptable size: +15
   - Too small/large: +5

4. **Experience Level** (0-15 points)
   - 5-10 years: +15
   - 3-5 years: +10
   - 10+ years: +10
   - <3 years: +5

5. **Location** (0-10 points)
   - Target location: +10
   - Acceptable location: +5
   - Remote: +8

**Total Score**: 0-100 points
- **90-100**: Hot lead - Contact immediately
- **70-89**: Warm lead - High priority follow-up
- **50-69**: Qualified - Standard outreach
- **<50**: Unqualified - Nurture or discard

### Automated Prospect Enrichment

Systematic enrichment workflow for large lists:

**Process**:
1. **Initial Search**: Get 100-500 prospects from LinkedIn search
2. **First Filter**: Remove obviously unqualified (wrong title, location, etc.)
3. **Batch Enrichment**: Enrich top 50 prospects with full profiles
4. **Email Discovery**: Find emails for top 25 prospects
5. **Web Research**: Extract company contacts for remaining prospects
6. **Final Scoring**: Apply lead scoring framework
7. **Export**: Generate CSV for top-scored leads only

**Efficiency Tips**:
- Start with larger searches (100+) to account for filtering
- Enrich in batches of 10-20 to avoid overwhelming results
- Prioritize email finding for highest-scored prospects first
- Use web scraping as backup when LinkedIn emails unavailable

## Limitations and Alternatives

### Platform Gaps

This skill focuses on B2B lead generation where LinkedIn, web scraping, and professional networks provide the most comprehensive data. For local business discovery and contact extraction, the combination of LinkedIn company search and web scraping provides strong coverage.

### Rate Limits and Timeouts

**Default Timeout**: 300 seconds (5 minutes) per MCP tool call

**Strategies for Large Datasets**:
1. **Batch Processing**: Use `count` parameter to limit results per call
   - Search 25-50 prospects at a time
   - Make multiple calls for larger lists
   - Allow processing time between batches

2. **Increase Timeout**: Use `request_timeout` parameter for complex queries
   - Company employee searches: 400-500 seconds
   - Profile enrichment: 300-400 seconds
   - Email finding: 400-600 seconds

3. **Parallel Processing**: For independent queries
   - Search multiple locations simultaneously
   - Enrich different prospect segments in parallel
   - Extract contacts from multiple websites at once

**Example**:
```
Instead of:
search_linkedin_users(count=200) → May timeout

Do this:
search_linkedin_users(count=50, location="California")
search_linkedin_users(count=50, location="New York")
search_linkedin_users(count=50, location="Texas")
search_linkedin_users(count=50, location="Florida")
```

### Data Freshness

**LinkedIn Data**: Real-time access through anysite MCP
- No caching (always current)
- Profile updates reflected immediately
- Company changes visible in real-time

**Email Database**: May have stale entries
- `get_linkedin_user_email_db` returns cached emails
- Some emails may be outdated
- Verify before mass outreach

**Recommendation**:
1. Use `find_linkedin_user_email` for current email discovery
2. Fall back to `get_linkedin_user_email_db` if first method fails
3. Validate emails before sending (use email validation service)
4. Update your CRM when emails bounce

### Privacy and Compliance

**LinkedIn Terms of Service**:
- Respect LinkedIn's usage policies
- Don't automate excessive requests
- Comply with user privacy settings
- Follow GDPR/CCPA requirements

**Email Collection**:
- Only collect publicly available emails
- Provide opt-out mechanisms
- Comply with anti-spam laws (CAN-SPAM, GDPR)
- Maintain suppression lists

**Best Practices**:
- Use data for legitimate business purposes only
- Respect connection request limits
- Don't scrape private profile information
- Maintain data security for collected leads

## Reference Documentation

For advanced techniques and strategies, see:

- **[LINKEDIN_STRATEGIES.md](references/LINKEDIN_STRATEGIES.md)** - Advanced LinkedIn search patterns, Boolean operators, and targeting strategies
- **[WEB_SCRAPING.md](references/WEB_SCRAPING.md)** - Website contact extraction patterns, common page structures, and parsing techniques

## Troubleshooting

### Common Issues

#### 1. No Results from LinkedIn Search

**Symptoms**: Search returns 0 results or very few

**Causes**:
- Overly restrictive search criteria
- Incorrect location format
- Title keywords too specific
- Company name misspelled

**Solutions**:
- Broaden search criteria (remove some filters)
- Try location variations: "San Francisco", "San Francisco Bay Area", "SF"
- Use partial titles: "Sales" instead of "Vice President of Sales"
- Verify company names with `search_linkedin_companies` first

**Example Fix**:
```
❌ Too Restrictive:
title="Vice President of Enterprise Sales"
location="San Francisco, California"
company_keywords="Salesforce Inc"

✅ Better:
title="VP Sales OR Head of Sales"
location="San Francisco Bay Area"
company_keywords="Salesforce"
```

#### 2. Email Not Found

**Symptoms**: `find_linkedin_user_email` returns no results

**Causes**:
- Email not in database
- User privacy settings
- Email verification required

**Solutions**:
1. Try `get_linkedin_user_email_db` as alternative
2. Extract email from company website instead
3. Use email pattern guessing (firstname.lastname@company.com)
4. Check for email in LinkedIn profile "Contact Info" section

**Alternative Workflow**:
```
1. get_linkedin_profile(user) → Get current company
2. get_linkedin_company(company) → Get company website
3. parse_webpage(website + "/contact") → Extract company emails
4. Use email pattern: first.last@companydomain.com
```

#### 3. Timeout Errors

**Symptoms**: Request times out after 300 seconds

**Causes**:
- Large result count requested
- Complex search criteria
- Server load issues

**Solutions**:
- Reduce `count` parameter (try 25-50 instead of 100+)
- Increase `request_timeout` to 400-600 seconds
- Break large searches into multiple smaller searches
- Simplify search criteria

**Example**:
```
❌ May Timeout:
search_linkedin_users(
  title="Software Engineer",
  count=500,
  request_timeout=300
)

✅ Better:
search_linkedin_users(
  title="Software Engineer",
  count=50,
  request_timeout=400
)
# Then make additional calls for more results
```

#### 4. Incomplete Profile Data

**Symptoms**: Profile missing work history, education, or skills

**Causes**:
- User profile not complete on LinkedIn
- Privacy settings limit data access
- Profile hasn't been updated

**Solutions**:
- Set `with_experience=true`, `with_education=true` explicitly
- Try getting detailed experience with `get_linkedin_user_experience`
- Accept incomplete data and supplement with web research
- Focus on prospects with complete profiles

#### 5. Invalid User URN

**Symptoms**: Error when using URN in follow-up calls

**Causes**:
- Incorrect URN format
- URN from different data source
- Stale URN reference

**Solutions**:
- Use LinkedIn username instead of URN when possible
- Verify URN format: `urn:li:fsd_profile:XXXXXXXXX`
- Get fresh URN from recent search
- Use profile URL as alternative identifier

### Getting Help

**anysite MCP Server Issues**:
- Verify MCP server is running and configured
- Check MCP server logs for errors
- Ensure authentication is set up correctly

**Skill-Specific Questions**:
- Review reference documentation in `references/` folder
- Check examples in this SKILL.md file
- Review common workflows section

**Data Quality Issues**:
- Validate search criteria before large batches
- Test with small `count` values first
- Review data quality in chat summary before export

**Integration Problems**:
- Verify CSV format matches your CRM requirements
- Test JSON structure before building integrations
- Check timezone and date formats in exports

## Example Use Cases

### Use Case 1: Enterprise SaaS Sales

**Goal**: Find 100 qualified enterprise sales prospects

**Process**:
1. Define ICP: VP/Director level, Enterprise Software, 500-5000 employees
2. Search LinkedIn: `title="VP Sales" company_keywords="Enterprise Software" count=100`
3. Filter by company size using `search_linkedin_companies`
4. Enrich top 30 prospects with full profiles
5. Find emails for top 20 prospects
6. Export CSV for Salesforce import

**Success Metrics**:
- 100 prospects found and qualified
- 30 enriched with full work history
- 15-20 verified emails obtained
- 2-3 hours total time investment

### Use Case 2: Tech Recruiter Sourcing

**Goal**: Source 50 Python engineers for startup

**Process**:
1. Search: `keywords="Python Django AWS" title="Software Engineer" location="Remote" count=100`
2. Filter: Remove <2 years experience
3. Enrich: Get skills and education for top 50
4. Score: Rank by skills match and experience level
5. Contact: Find emails for top 25
6. Outreach: Personalized emails referencing their skills/projects

**Success Metrics**:
- 50 qualified candidates sourced
- 25 emails obtained
- 5-10 candidates engaged
- Time to first response: <24 hours

### Use Case 3: Partnership Outreach

**Goal**: Find potential integration partners in marketing tech

**Process**:
1. Search companies: `keywords="marketing automation" employee_count=["51-200"]`
2. Identify decision-makers: `title="VP Product OR Head of Partnerships" company_keywords=<company>`
3. Research: Get company details and recent LinkedIn posts
4. Enrich: Full profiles for all decision-makers
5. Personalize: Reference their product and use cases
6. Multi-channel: LinkedIn + email outreach

**Success Metrics**:
- 30 potential partners identified
- 40 decision-makers found
- 25 emails + LinkedIn connection requests sent
- 10-15 responses received

---

**Ready to start generating leads?** Ask Claude to help you find prospects, enrich profiles, or build comprehensive lead lists using this skill!
