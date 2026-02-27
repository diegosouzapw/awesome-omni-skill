---
name: add-event
description: Add an event/conference to your CV's relevant events list with intelligent data extraction, validation, and automatic commit generation
---

# Add Event to CV

Add an event or conference to your CV's relevant events list with intelligent data extraction and validation.

## Usage

```
/add-event <event identifier>
```

Where `<event identifier>` can be:

- Event title (e.g., "AWS re:Invent 2024")
- URL to event page (Eventbrite, Meetup, Luma, conference websites, etc.)
- Path to screenshot containing event information
- Leave empty to enter manual mode with prompts

## Behavior

When invoked, this skill will:

### 1. **Extract Event Information**

   - If a URL is provided: Fetch and parse the page for title, location, and date
   - If a screenshot is provided: Parse the image using vision capabilities to extract event details
   - If a title is provided: Search the web for the event to get accurate location and date
   - If empty: Prompt user for title, location, and date manually

### 2. **Validate the Event**

   - Confirm the event exists and information is accurate
   - Check for duplicates in the existing `relevantEvents` array
   - Alert if the event is already present
   - Validate date and location formats

### 3. **Format the Entry**

   Properly format the event entry following existing conventions:

   - **Location format**: "City, Country" or "Virtual"
     - Examples: "San Francisco, USA", "Berlin, Germany", "Virtual"
     - Standardize country names (USA not US, etc.)
   
   - **Date format**:
     - Single day: "DD MMM YYYY" (e.g., "21 Oct 2015")
     - Multi-day same month: "DD-DD MMM YYYY" (e.g., "27-28 Aug 2024")
     - Multi-day cross-month: "DD MMM - DD MMM YYYY" (e.g., "30 Jan - 2 Feb 2024")
   
   - **Title**: Use official event name, preserve capitalization and special characters

   Example formats:

   ```typescript
   {
       title: 'Engineering Leadership Community Annual 2024',
       location: 'San Francisco, USA',
       date: '27-28 Aug 2024',
   }

   {
       title: 'TEDx Goiânia',
       location: 'Goiânia, Brazil',
       date: '21 Oct 2015',
   }

   {
       title: 'ELC Match & Learn: increasing velocity without losing quality',
       location: 'Virtual',
       date: '25 Apr 2024',
   }
   ```

### 4. **Determine Insertion Position**

   - Parse existing event dates to determine chronological position
   - Events are sorted newest first (reverse chronological order)
   - Insert in the correct position to maintain order
   - Show where in the list it will be inserted

### 5. **Show Proposed Changes**

   - Display the complete diff showing:
     - The formatted event entry
     - Its position in the list (with before/after context)
     - Line numbers for reference
   - Highlight that events are in reverse chronological order

### 6. **Request User Approval**

   - Present the changes and ask for explicit confirmation
   - Options: "approve", "edit", "cancel"
   - If "edit" is selected, allow user to modify title, location, or date

### 7. **Commit Changes**

   - If approved, use the Edit tool to add the event to `src/components/cv/cvData.ts`
   - Create a descriptive commit message:

     ```
     Add event to CV: [Event Title]

     Location: [Location]
     Date: [Date]

     Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
     ```

   - Run `npm run lint` to ensure code formatting is correct

### 8. **Offer to Push**
   - After successful commit, ask if the user wants to push to the remote branch
   - If yes, execute: `git push -u origin <branch-name>`

## Important Notes

- **File Location**: Events are stored in `src/components/cv/cvData.ts`
- **Structure**: Single array at `cvData.relevantEvents[]` (around lines 797-850)
- **Ordering**: Events MUST be in reverse chronological order (newest first)
- **Date Parsing**:
  - Accept various input formats (MM/DD/YYYY, DD-MM-YYYY, "Month DD, YYYY", etc.)
  - Always convert to CV format: "DD MMM YYYY" or "DD-DD MMM YYYY"
  - Use 3-letter month abbreviations (Jan, Feb, Mar, etc.)
- **Location Standardization**:
  - Always use full country names: "USA" not "US"
  - Format: "City, Country" - capitalize appropriately
  - Use "Virtual" for online-only events (not "Online", "Remote", etc.)
- **Linting**: Always run `npm run lint` before committing to maintain code quality
- **PDF Generation**: Remind user they may want to rebuild the CV PDF with `npm run build`

## Supported Event Platforms

The skill can parse event information from:

- Eventbrite (eventbrite.com)
- Meetup (meetup.com)
- Luma (lu.ma)
- LinkedIn Events
- Conference websites (via HTML/JSON-LD parsing)
- Generic event pages with structured data

## Error Handling

- If event information cannot be found or validated, ask user to provide details manually
- If duplicate is detected, inform user and ask if they want to continue anyway
- If date format is ambiguous, ask user to clarify
- If location format is unclear, suggest standard format and confirm
- If web search/URL fetch fails, fall back to manual entry mode

## Examples

```bash
# Add by title (will search for event details)
/add-event AWS re:Invent 2024

# Add by URL
/add-event https://www.eventbrite.com/e/example-event-tickets-123456

# Add by Luma URL
/add-event https://lu.ma/example-event

# Add by screenshot
/add-event /path/to/event-screenshot.png

# Manual mode (prompts for all details)
/add-event
```

## Date Format Examples

Input formats accepted:
- "August 27-28, 2024"
- "27-28 Aug 2024"
- "2024-08-27 to 2024-08-28"
- "27/08/2024 - 28/08/2024"
- "Aug 27, 2024"

Output formats (standardized):
- Single day: "21 Oct 2015"
- Multi-day (same month): "27-28 Aug 2024"
- Multi-day (cross month): "30 Jan - 2 Feb 2024"

## Configuration

Default behavior can be customized:

- **auto-push**: Set to `true` to automatically push after commit (default: `false`)
- **auto-approve**: Set to `true` to skip approval step (default: `false`, not recommended)
- **date-format-preference**: Preferred input date format for ambiguous dates (default: `auto-detect`)
- **sort-order**: Maintain chronological order (default: `newest-first`, do not change)

## Workflow Tips

1. **Batch Adding**: If adding multiple events, you can invoke the skill multiple times in succession
2. **Recent Events**: When adding recent events, dates are usually easier to find and validate
3. **Historical Events**: For older events, you may need to rely on manual entry if details are not online
4. **Virtual Events**: Always use "Virtual" as the location for online-only events
5. **Event Series**: For recurring events (e.g., "Annual Conference 2024"), include the year in the title

## Related Commands

- `npm run dev` - Preview changes locally
- `npm run build` - Build production site and generate updated PDF
- `npm run lint` - Format code (automatically run by skill)
- `git log` - View commit history

## Troubleshooting

**Problem**: Date format not recognized
- **Solution**: Provide date in "DD MMM YYYY" format directly or let the skill prompt for clarification

**Problem**: Event marked as duplicate but seems different
- **Solution**: Check if the event title is slightly different (e.g., year included or not)

**Problem**: Location format incorrect
- **Solution**: Use "City, Country" format exactly, or "Virtual" for online events

**Problem**: Commit fails
- **Solution**: Check that you're on a clean branch and have no uncommitted changes

## Testing Checklist

Before considering the skill complete, verify:

- [ ] Can add event by title (web search)
- [ ] Can add event by URL (multiple platforms)
- [ ] Can add multi-day event with correct date format
- [ ] Can add single-day event
- [ ] Can add virtual event
- [ ] Duplicate detection works
- [ ] Events inserted in correct chronological position (newest first)
- [ ] Date format conversion works for various inputs
- [ ] Location standardization works
- [ ] Commit message generated correctly
- [ ] Linting runs successfully
- [ ] Changes show correct diff with context
