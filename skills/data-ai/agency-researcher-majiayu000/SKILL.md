---
name: agency-researcher
description: Find and qualify real estate agencies in a given suburb
---

# Agency Researcher Skill

You are an expert real estate industry researcher. Find, analyze, and qualify real estate agencies in Australian suburbs.

## Mission

Given a suburb name, find top real estate agencies and gather comprehensive data for each. This data powers personalized demo pages.

## Research Process

### Step 1: Initial Search

Search for agencies using queries like:
- "[suburb] real estate agents"
- "[suburb] real estate agencies"
- "best real estate agents [suburb] Sydney"

Use WebSearch. Aim for 8-12 agencies initially.

### Step 2: For Each Agency, Gather Data

Visit each agency website via Chrome browser tools. Extract:

**Basic Info**
- Agency Name (official business name)
- Website URL
- Phone Number (header, footer, contact page)
- Email (general enquiry)
- Address (physical office)

**Branding (CRITICAL)**
- Logo URL: Direct image URL (.png/.svg/.jpg), publicly accessible
- Primary Brand Color: Main color (hex code from header/buttons/headings)
- Secondary Color: Accent color (hex code from hover states/borders)

**Team & Size**
- Team Size: Count agents on "Our Team" page
- Principal/Owner Name: Look for "Principal", "Director", "Owner"

**Listing Activity**
- Active Listings Count: From properties/listings page
- Has Property Management: Check services for PM/Rentals/Landlords (boolean)

**Pain Indicators**
- Has After-Hours Number: Check for "after hours", "24/7" (boolean)
- Has Chat Widget: Look for Intercom/Drift/LiveChat bubble (boolean)
- Has Online Booking: Inspection booking on listing pages (boolean)

**Reviews (optional)**
- Search "[Agency Name] reviews"
- Note any "couldn't reach", "didn't answer" mentions

### Step 3: Calculate Pain Score (0-100)

```
Base: 0

+20 if 30+ listings
+15 if 20-29 listings
+10 if 10-19 listings

+25 if has Property Management

+20 if <5 agents AND 20+ listings
+15 if <3 agents AND 10+ listings

+15 if no after-hours number
+10 if no chat widget
+5 if no online booking
+10 if bad review signals
```

### Step 4: Generate Pain Reasons

Create specific pain points list:
- "45 active listings generating high call volume"
- "Team of only 4 agents managing 30+ properties"
- "No after-hours contact solution"

### Step 5: Output Format

Save to the absolute output path provided in the prompt (suburb results JSON):

```json
{
  "suburb": "Surry Hills",
  "searchedAt": "2025-01-15T10:30:00Z",
  "agencies": [
    {
      "id": "ray-white-surry-hills",
      "name": "Ray White Surry Hills",
      "website": "https://raywhitesurryhills.com.au",
      "phone": "+61 2 9361 6000",
      "email": "surryhills.nsw@raywhite.com",
      "address": "123 Crown Street, Surry Hills NSW 2010",
      "branding": {
        "logoUrl": "https://raywhitesurryhills.com.au/logo.png",
        "primaryColor": "#ffe512",
        "secondaryColor": "#1a1a1a"
      },
      "metrics": {
        "teamSize": 6,
        "listingCount": 45,
        "hasPropertyManagement": true,
        "hasAfterHoursNumber": false,
        "hasChatWidget": false,
        "hasOnlineBooking": true,
        "principalName": "John Smith"
      },
      "painScore": 87,
      "painReasons": [
        "45 active listings generating high call volume",
        "No after-hours contact solution"
      ],
      "notes": "Major franchise"
    }
  ]
}
```

Also save individual files to absolute paths under the project’s `data/agencies/` directory.

## Quality Guidelines

- Logo URL must be direct image URL, publicly accessible
- Use hex values for colors, not color names
- Phone numbers: +61 or 02/03/07/08 format
- Use null for missing data, not empty string
- Don't make up data
