---
name: athlete-docs
description: Documentation guidance for competition athletes and volunteers in WODsmith. Use when writing, reviewing, or improving athlete-facing documentation including registration, scheduling, workout viewing, leaderboards, check-in, and volunteer coordination.
---

# Athlete Documentation Skill

Documentation guidance for the athlete persona in WODsmith competition platform.

## Athlete Persona

**Who they are:** CrossFit athletes registering for and participating in competitions, and volunteers helping at events.

**Key differences from organizers:**

| Aspect | Athletes | Organizers |
|--------|----------|------------|
| Interaction | Consumer (read-heavy) | Producer (write-heavy) |
| Complexity | Simple workflows | Complex multi-step setup |
| Discovery | High (browsing, exploring) | Low (knows what they need) |

## Critical Workflow

**REQUIRED**: Before writing athlete docs:

1. Load the parent skill: `skills_use(name="documentation")`
2. Apply the Diataxis compass from that skill
3. Load the workflow reference below for athlete-specific guidance

## Workflow References (load on demand)

**REQUIRED**: Before writing docs, load the relevant reference:

| If documenting... | Load this reference |
|-------------------|---------------------|
| Registration, Division selection, Confirmation | `references/registration-journey.md` |
| Schedule, Heats, Workouts, Check-in | `references/competition-day.md` |
| Leaderboard, Scores, Final standings | `references/results-tracking.md` |
| Volunteer signup, Credentials, Assignments | `references/volunteer-journey.md` |

## Documentation Mapping

### Tutorials (3 workflows)
- Register for Your First Competition
- Volunteer at an Event  
- Track Your Results During Competition

### How-to Guides (6 workflows)
- Update Your Registration
- View Your Heat Schedule
- Check Your Division Standings
- View Workout Standards
- Complete Event Check-in
- Change Division Selection

### Reference (4 topics)
- Scoring Rules
- Division Requirements
- Heat Format Specifications
- Registration Status Definitions

### Explanation (4 topics)
- How Heats Work
- Scoring Methodology
- The Division System
- Competition Day Flow

## User Journeys

**Journey 1: Discovery to Registration**
Find Competition → Explore Details → Register → Confirm

**Journey 2: Pre-Competition Prep**
Check Schedule → View Workouts → Plan Strategy

**Journey 3: Competition Day**
Check-in → View Heat Schedule → Track Scores

**Journey 4: Post-Competition**
View Results → Compare Rankings → Share

**Journey 5: Volunteer Experience**
Sign Up → Get Assignment → Event Day

## Route Reference

| Route | Auth | Primary Doc Type |
|-------|------|------------------|
| `/c/[slug]` | No | N/A (entry point) |
| `/c/[slug]/register` | Yes | Tutorial / How-to |
| `/c/[slug]/volunteer` | No | Tutorial / How-to |
| `/c/[slug]/workouts` | No | Reference |
| `/c/[slug]/schedule` | No | How-to |
| `/c/[slug]/leaderboard` | No | How-to / Tutorial |
| `/my-schedule` | Yes | How-to |
