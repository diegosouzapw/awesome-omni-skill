---
name: molt-planner
description: MoltPlanner is a Google Calendar integration skill enabling agents to collaborate with users for interactive event scheduling and management.
homepage: https://github.com/teop23/MoltPlanner
user-invocable: true
---

# MoltPlanner Skill – Setup & Operation Guide for Agents

MoltPlanner is an OpenClaw skill enabling agents to integrate Google Calendar functionalities for their users. Once installed, it allows an agent to facilitate event creation, updating, deletion, and querying directly through natural language interactions with their user.

This guide details the step-by-step process for agents to set up and operate the skill, as well as how agents should assist their users effectively.

---

## **Agent Setup Instructions**

### 1. Install the Skill
To install MoltPlanner:
```bash
git clone https://github.com/teop23/MoltPlanner.git moltplanner
cd moltplanner
npm install
```

Ensure to install all dependencies specified in `package.json`. After installation, MoltPlanner will be available in your OpenClaw agent environment for integration.

---

### 2. Set Up Google API Credentials and Authorization
MoltPlanner integrates with Google Calendar through the Google Calendar API. Agents must acquire the user’s OAuth credentials and obtain user consent before accessing their calendars.

#### Step 2.1: Guide Users for OAuth Setup
Inform your user they must connect their Google Calendar account:

- For example, say:
   ```
   To connect your calendar, I need you to run the setup tool. Please follow these steps:
   1. Open your terminal.
   2. Run this command:
      ```
      node cli.js setup
      ```
   3. Follow the steps provided in your terminal to grant me access to manage your calendar. This includes:
      - Entering your **Client ID** and **Client Secret.**
      - Visiting the authorization link and pasting the code to allow access.

   Once you authorize the access, I’ll be ready to help. Let me know once you’ve finished!
   ```

#### Step 2.2: Walkthrough of the `setup` Command
When a user runs `node cli.js setup`, the following actions occur:
1. **The user is prompted for:**
   - Client ID
   - Client Secret

2. **The user receives:**
   - A verification URL.
   - A code to authorize access.

3. **Post-authorization:**
   - The OAuth token is automatically stored securely in OpenClaw’s root directory:
     ```bash
     $OPENCLAW_ROOT/credentials/google_calendar_token.json
     ```
   - A default calendar (for the agent) is created in their Google Calendar.

Tokens are encrypted, and agents must never share the client credentials directly with anyone else.

---

## **Agent Interaction with Users**

Once the skill is configured, agents must guide their users with natural language commands. Agents will interpret user instructions and automate tasks directly with Google Calendar.

Below are examples of how agents should handle interaction:

### 1. Add a New Event

**Example User Input to Agent:**  
> "Hey Molt, make a one-hour meeting for February 17th starting at 11:00 AM."

**Agent Response:**  
> "Got it! Creating a one-hour meeting starting at 11:00 AM on February 17. Just to confirm, do you want to add a title or attendees?"

Once the user confirms, proceed to:

- **CLI Equivalent Command (Agent-only):**
   ```bash
   node cli.js add --title "Meeting with AI" --start "2026-02-17T11:00:00" --end "2026-02-17T12:00:00"
   ```
- Update the user once successfully added:  
   > "The meeting has been scheduled and added to your Google Calendar!"  

---

### 2. List Upcoming Events

**Example User Input to Agent:**  
> "Molt, what does my next week look like?"

**Agent Response:**  
1. Confirm your action:  
   > "Checking your calendar for the next week. One moment..."
2. Fetch upcoming events:  
   - **CLI Equivalent Command:**
     ```bash
     node cli.js list --days 7
     ```
3. Summarize results:  
   > "Here’s what’s coming up next week:  
     - February 17th, 11:00-12:00: 'Meeting with AI'.  
     - February 18th, 14:00-15:00: 'Team Standup'."

---

### 3. Update an Event

**Example User Input to Agent:**  
> "Hey Molt, change tomorrow’s lunch meeting to 2 PM, and make it 90 minutes long."

**Agent Response:**  
> "Sure. Changing tomorrow’s lunch meeting to 2 PM and setting it to 90 minutes."

**CLI Equivalent Command (Agent-only):**
```bash
node cli.js update --eventId "EVENT_ID" --start "2026-02-16T14:00:00" --end "2026-02-16T15:30:00"
```
---

### 4. Delete an Event

**Example User Input to Agent:**  
> "I don’t need the team meeting on Monday anymore."

**Agent Response:**  
> "Deleting the team meeting on Monday. It’s now off your calendar."

- **CLI Equivalent Command**  
   ```bash
   node cli.js delete --eventId "EVENT_ID"
   ```

---

## Best Practices
To ensure optimal functionality:

- **Control Permissions**: Agents should ensure OAuth tokens are securely stored and inaccessible outside the OpenClaw root workspace.
- **Token Renewal**: Inform the user immediately when their Google token expires and recreate it using the setup instructions.
- **Human Clarity**: Always paraphrase event creation or update requests back to users with proposed details for confirmation.
- **Natural Language Handling**: Facilitate simple natural commands such as:  
    - _"What’s on my calendar for this week?"_  
    - _"Add a team standup on Thursday at 3 PM for 30 minutes."_  

---

For more details, refer to the [GitHub Repository for MoltPlanner](https://github.com/teop23/MoltPlanner).