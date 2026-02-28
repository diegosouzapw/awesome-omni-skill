---
name: linear-dev-accelerator
description: Accelerate software development with Linear project management and MCP server integration. Master issue tracking, project workflows, and development automation for frontend, full-stack, and mobile applications. Includes comprehensive MCP tool usage, workflow patterns, and development best practices.
---

# Linear Development Accelerator

A comprehensive skill for accelerating software development using Linear project management with MCP server integration. This skill enables rapid project setup, intelligent issue management, and streamlined development workflows for web and mobile applications.

## When to Use This Skill

Use this skill when:

- Starting a new software development project and need to set up Linear project management
- Managing issues, tasks, and features for frontend, backend, or full-stack applications
- Organizing development work into sprints/cycles with proper tracking
- Collaborating with development teams on software projects
- Automating project management tasks during development
- Tracking bugs, features, and technical debt systematically
- Building applications with structured project management workflows
- Integrating Linear with GitHub for PR-based development

## Core Concepts

### Linear Project Management Philosophy

Linear emphasizes speed, clarity, and focus for software teams:

- **Issues as First-Class Citizens**: Every unit of work is an issue with rich metadata
- **Cycles for Sprint Planning**: Time-boxed iterations for focused execution
- **Projects for Product Planning**: Higher-level initiatives and roadmaps
- **Teams for Organization**: Workspaces for different functions or products
- **Workflows for Process**: Custom statuses matching your development process

### Key Linear Entities

1. **Teams**: Organizational units (e.g., Frontend, Backend, Mobile)
2. **Issues**: Individual tasks, bugs, features, or improvements
3. **Projects**: Collections of related issues for larger initiatives
4. **Cycles**: Time-boxed iterations (similar to sprints)
5. **Labels**: Categorization and filtering mechanism
6. **Statuses**: Issue states (Backlog, Todo, In Progress, In Review, Done, Canceled)
7. **Comments**: Collaboration and discussion on issues
8. **Documents**: Long-form documentation integrated with issues

## MCP Server Tools Reference

### Issue Management

#### list_issues
List and filter issues in your workspace.

**Common Parameters:**
```
team: Team name or ID
assignee: User ID, name, email, or "me" for your issues
state: Status name or ID (e.g., "In Progress", "Done")
label: Label name or ID for filtering
project: Project name or ID
query: Search in title/description
limit: Number of results (max 250, default 50)
orderBy: "createdAt" or "updatedAt" (default)
includeArchived: Include archived issues (default: true)
```

**Examples:**
```
# Get my current issues
assignee: "me"
state: "In Progress"
limit: 10

# Find bugs in a specific project
label: "bug"
project: "Mobile App"
state: "Todo"

# Search for authentication-related issues
query: "authentication"
team: "Backend"
```

#### create_issue
Create a new issue with full metadata.

**Required Parameters:**
```
title: Issue title (clear and concise)
team: Team name or ID
```

**Optional Parameters:**
```
description: Markdown description with details
assignee: User ID, name, email, or "me"
state: State type, name, or ID
priority: 0=None, 1=Urgent, 2=High, 3=Normal, 4=Low
labels: Array of label names or IDs
project: Project name or ID
cycle: Cycle name, number, or ID
parentId: Parent issue ID for sub-issues
dueDate: ISO format date
links: Array of {url, title} objects
```

**Example:**
```
title: "Implement user authentication"
team: "Backend"
description: |
  ## Overview
  Add JWT-based authentication to the API

  ## Tasks
  - [ ] Design auth flow
  - [ ] Implement JWT tokens
  - [ ] Add refresh token logic
  - [ ] Write tests

  ## Acceptance Criteria
  - Secure token generation
  - Token expiration handling
  - Proper error responses
assignee: "me"
priority: 2  # High
labels: ["feature", "authentication", "backend"]
project: "API v2.0"
```

#### update_issue
Update an existing issue.

**Required:** `id` (issue ID)

**Updatable Fields:**
```
title, description, assignee, state, priority, labels,
project, cycle, parentId, dueDate, estimate, links
```

**Example:**
```
id: "issue-uuid-here"
state: "In Progress"
assignee: "me"
labels: ["feature", "in-development"]
```

#### get_issue
Get detailed issue information including attachments and git branch.

**Parameters:**
```
id: Issue ID
```

**Returns:** Full issue details with attachments, git branch name, related data

### Project Management

#### list_projects
List and filter projects.

**Parameters:**
```
team: Team name or ID
state: State name or ID
member: User ID, name, email, or "me"
initiative: Initiative name or ID
query: Search project names
limit: Number of results (max 250, default 50)
orderBy: "createdAt" or "updatedAt"
includeArchived: Include archived (default: false)
```

#### create_project
Create a new project.

**Required Parameters:**
```
name: Project name
team: Team name or ID
```

**Optional Parameters:**
```
description: Full Markdown description
summary: Concise plaintext summary (max 255 chars)
lead: Project lead (User ID, name, email, or "me")
state: Project state
priority: 0-4 (None, Urgent, High, Medium, Low)
startDate: ISO format date
targetDate: ISO format date
labels: Array of label names or IDs
```

**Example:**
```
name: "Mobile App v2.0"
team: "Mobile"
summary: "Complete redesign of mobile app with new features"
description: |
  # Mobile App v2.0

  ## Goals
  - Modern UI/UX redesign
  - Offline-first architecture
  - Push notifications
  - Real-time sync

  ## Timeline
  Q1 2025 - Design & Planning
  Q2 2025 - Development
  Q3 2025 - Testing & Launch
lead: "me"
priority: 2
startDate: "2025-01-15"
targetDate: "2025-09-30"
labels: ["mobile", "redesign", "major-release"]
```

#### update_project
Update existing project details.

**Required:** `id` (project ID)

**Updatable:** name, description, summary, lead, state, priority, startDate, targetDate, labels

#### get_project
Get detailed project information.

**Parameters:**
```
query: Project ID or name
```

### Labels & Categorization

#### list_issue_labels
List available labels.

**Parameters:**
```
team: Team name or ID (optional, for team-specific labels)
name: Filter by label name
limit: Number of results (max 250)
orderBy: "createdAt" or "updatedAt"
```

#### create_issue_label
Create new label for categorization.

**Required:**
```
name: Label name
```

**Optional:**
```
color: Hex color code (e.g., "#10B981")
description: Label description
teamId: Team UUID (omit for workspace label)
isGroup: true for label groups (default: false)
parentId: Parent label UUID for hierarchical labels
```

**Example:**
```
name: "frontend"
color: "#3B82F6"
description: "Frontend-related tasks and features"
teamId: "team-uuid-here"
```

### Workflow & Status Management

#### list_issue_statuses
List available statuses for a team.

**Parameters:**
```
team: Team name or ID (required)
```

**Returns:** Array of statuses with id, type, name
- Types: backlog, unstarted, started, completed, canceled

#### get_issue_status
Get detailed status information.

**Parameters:**
```
id: Status ID
name: Status name
team: Team name or ID (required)
```

### Collaboration

#### list_comments
List comments on an issue.

**Parameters:**
```
issueId: Issue ID (required)
```

#### create_comment
Add comment to an issue.

**Parameters:**
```
issueId: Issue ID (required)
body: Markdown comment content (required)
parentId: Parent comment ID for replies (optional)
```

**Example:**
```
issueId: "issue-uuid-here"
body: |
  Great progress! A few notes:

  - Consider edge case for expired tokens
  - Add integration test for refresh flow
  - Document the auth header format
```

### Team & User Management

#### list_teams
List all teams in workspace.

**Parameters:**
```
query: Search query (optional)
limit: Number of results (max 250)
includeArchived: Include archived teams (default: false)
```

#### get_team
Get detailed team information.

**Parameters:**
```
query: Team UUID, key, or name
```

#### list_users
List users in workspace.

**Parameters:**
```
query: Filter by name or email (optional)
```

#### get_user
Get user details.

**Parameters:**
```
query: User ID, name, email, or "me"
```

### Cycles (Sprint Planning)

#### list_cycles
Get cycles for a team.

**Parameters:**
```
teamId: Team ID (required)
type: "current", "previous", "next", or omit for all
```

### Documentation

#### list_documents
List Linear documents.

**Parameters:**
```
query: Search query
creatorId: Creator user ID
projectId: Project ID
initiativeId: Initiative ID
limit: Number of results (max 250)
orderBy: "createdAt" or "updatedAt"
includeArchived: Include archived (default: false)
```

#### get_document
Get document by ID or slug.

**Parameters:**
```
id: Document ID or slug
```

## Development Workflow Patterns

### Pattern 1: New Feature Development

**Scenario:** Building a new feature for a web application

```workflow
1. Create feature issue
   - Use create_issue with:
     - Clear title: "Add dark mode toggle"
     - Detailed description with tasks and acceptance criteria
     - Assign to yourself or team member
     - Add labels: ["feature", "frontend", "ui"]
     - Link to project: "Q1 Features"
     - Set priority based on urgency

2. Break down into sub-tasks (if complex)
   - Create child issues with parentId
   - Each sub-task is a manageable unit of work
   - Example sub-tasks:
     - "Design dark mode color scheme"
     - "Implement theme context in React"
     - "Add toggle button component"
     - "Persist theme preference"

3. Track progress
   - Update issue state as you work:
     - Todo → In Progress → In Review → Done
   - Add comments with updates and blockers
   - Link PR when ready for review

4. Complete and review
   - Update state to "In Review"
   - Add links to PR, deployed preview
   - Mark as Done when merged and deployed
```

### Pattern 2: Bug Triage and Fix

**Scenario:** Handling bug reports efficiently

```workflow
1. Create bug issue
   - title: Clear description of the bug
   - description: Steps to reproduce, expected/actual behavior
   - labels: ["bug", "priority:high"] if critical
   - assignee: Developer responsible for that area
   - Add links to error logs, screenshots

2. Investigation
   - Update to "In Progress"
   - Add comments with findings
   - Link to related issues if discovered
   - Update priority if needed

3. Fix implementation
   - Reference issue in commit messages
   - Link PR to issue
   - Add comment explaining fix approach

4. Verification and closure
   - State: "In Review" during QA
   - State: "Done" after verification
   - Add comment confirming fix is deployed
```

### Pattern 3: Sprint/Cycle Planning

**Scenario:** Planning a 2-week development cycle

```workflow
1. Review backlog
   - list_issues with state: "Backlog"
   - Filter by priority and labels
   - Identify issues ready for development

2. Create or select cycle
   - list_cycles to see current cycle
   - Organize selected issues into cycle

3. Assign and prioritize
   - Distribute issues across team
   - Set priorities: Urgent (1) → High (2) → Normal (3)
   - Balance workload across team members

4. Daily tracking
   - list_issues with assignee: "me", state: "In Progress"
   - Update issue states as work progresses
   - Add comments for standup updates
   - Move blocked issues back to Todo

5. Cycle review
   - List completed issues in cycle
   - Review metrics (velocity, completion rate)
   - Move incomplete issues to next cycle or backlog
```

### Pattern 4: Project Setup for New Application

**Scenario:** Starting a new mobile app project

```workflow
1. Create project
   - name: "Mobile App - iOS & Android"
   - description: Full project scope, goals, timeline
   - Set lead, dates, labels
   - Link to design docs, PRD

2. Set up project structure
   - Create initial issues for setup:
     - "Setup React Native project"
     - "Configure CI/CD pipeline"
     - "Design component architecture"
     - "Setup state management"
   - Assign to team members
   - Add to project

3. Create epics (major features)
   - Authentication system
   - User profile management
   - Core app functionality
   - Push notifications
   - Each epic is a parent issue with sub-issues

4. Organize into cycles
   - Cycle 1: Project setup & architecture
   - Cycle 2-3: Core features
   - Cycle 4: Polish & testing
   - Assign issues to appropriate cycles

5. Track progress
   - Update project description with status
   - Weekly reviews with get_project
   - Adjust priorities and timeline as needed
```

### Pattern 5: GitHub Integration Workflow

**Scenario:** Linking Linear issues with GitHub PRs

```workflow
1. Create issue in Linear
   - Get git branch name from issue: issue.gitBranchName
   - Example: "cetiaiservices/cet-95-create-benchmark-methodology-guide"

2. Create branch using Linear's suggested name
   - Ensures automatic linking
   - Linear tracks PR status automatically

3. Development
   - Commit with Linear issue reference: "CET-95: Add benchmark docs"
   - Linear updates issue automatically when PR created

4. Code review
   - PR opened → Linear sets state to "In Review"
   - Comments on PR can sync to Linear
   - Review feedback tracked in both systems

5. Merge and deploy
   - PR merged → Linear can auto-complete issue
   - Deployment triggers update Linear with links
   - Full traceability from issue → PR → deployment
```

## Accelerated Development Strategies

### Strategy 1: Batch Issue Creation

When starting a project, create all known issues in one go:

```example
// Create multiple related issues
const features = [
  "User registration and login",
  "Password reset flow",
  "Email verification",
  "Social auth (Google, GitHub)",
  "Two-factor authentication"
];

// For each feature, create issue with:
- Consistent labeling: ["auth", "feature"]
- Link to parent project: "Authentication System"
- Assign to team members based on expertise
- Set due dates based on project timeline
```

### Strategy 2: Smart Filtering for Focus

Use filters to reduce cognitive load:

```example
// Morning: What do I need to work on today?
assignee: "me"
state: "In Progress"
orderBy: "updatedAt"

// Planning: What's in the backlog for my team?
team: "Frontend"
state: "Backlog"
priority: 2  // High priority items
labels: ["ready-for-dev"]

// Review: What needs my attention?
assignee: "me"
state: "In Review"
orderBy: "createdAt"  // Oldest first
```

### Strategy 3: Template-Based Issue Creation

Create consistent issues using templates:

```example
// Bug report template
title: "[BUG] {concise description}"
description: |
  ## Bug Description
  {What's wrong?}

  ## Steps to Reproduce
  1. {Step 1}
  2. {Step 2}
  3. {Step 3}

  ## Expected Behavior
  {What should happen}

  ## Actual Behavior
  {What actually happens}

  ## Environment
  - Browser: {browser}
  - OS: {OS}
  - Version: {version}

  ## Additional Context
  {Screenshots, logs, etc.}
labels: ["bug", "needs-triage"]
priority: 3  // Default to normal, adjust after triage

// Feature request template
title: "[FEATURE] {feature name}"
description: |
  ## Feature Description
  {What feature do you want?}

  ## User Story
  As a {user type}
  I want {goal}
  So that {benefit}

  ## Acceptance Criteria
  - [ ] {Criterion 1}
  - [ ] {Criterion 2}
  - [ ] {Criterion 3}

  ## Design Notes
  {Link to designs, wireframes}

  ## Technical Notes
  {Implementation considerations}
labels: ["feature", "needs-refinement"]
```

### Strategy 4: Automated Status Updates

Update issues as part of your development workflow:

```example
// In CI/CD pipeline:
- Tests passing → Add comment with test results
- Deploy to staging → Update with staging URL
- Deploy to production → Mark as Done, add production URL

// In git hooks:
- Pre-commit: Verify issue reference in commit message
- Post-merge: Update Linear issue status
- Tag creation: Link release to completed issues
```

### Strategy 5: Cross-Functional Coordination

Use Linear for coordinating frontend/backend/mobile work:

```example
// Create parent issue for full-stack feature
Parent: "User Dashboard Feature"
  ├─ Child: "[Backend] Dashboard API endpoints"
  │   └─ labels: ["backend", "api"]
  ├─ Child: "[Frontend] Dashboard React components"
  │   └─ labels: ["frontend", "react"]
  └─ Child: "[Mobile] Dashboard mobile screens"
      └─ labels: ["mobile", "flutter"]

// Track dependencies
- Backend issue must complete first
- Frontend/Mobile can work in parallel after API ready
- Use "blocked" relation type to track dependencies
```

## Best Practices

### Issue Creation

1. **Clear Titles**: Start with verb (Add, Fix, Update, Refactor)
2. **Rich Descriptions**: Use markdown, checklists, code blocks
3. **Proper Labeling**: Consistent label taxonomy
4. **Right Sizing**: Issues should be completable in 1-3 days
5. **Acceptance Criteria**: Always define "done"

### Project Organization

1. **One Project Per Initiative**: Don't create too many projects
2. **Use Labels for Categorization**: More flexible than projects
3. **Regular Cleanup**: Archive completed projects
4. **Clear Ownership**: Every project needs a lead
5. **Linked Documentation**: Keep project descriptions updated

### Workflow Management

1. **Limit Work in Progress**: Max 3 "In Progress" issues per person
2. **Update Status Promptly**: Keep issues current
3. **Use Comments Liberally**: Document decisions and blockers
4. **Link Related Issues**: Build context through relations
5. **Complete Issues Fully**: Don't leave zombie issues

### Team Collaboration

1. **Assign Clearly**: Every issue should have one owner
2. **Use @mentions**: Tag people in comments for visibility
3. **Regular Reviews**: Weekly project/cycle reviews
4. **Celebrate Wins**: Mark milestones and achievements
5. **Share Context**: Link to PRs, docs, designs

### Label Taxonomy

Recommended label categories:

```
Type Labels:
- feature, bug, improvement, docs, refactor, test

Priority Labels:
- priority:urgent, priority:high, priority:normal, priority:low

Area Labels:
- frontend, backend, mobile, devops, design, qa

Status Labels:
- blocked, ready-for-dev, needs-review, waiting-on-qa

Technology Labels:
- react, flutter, python, node, postgres, redis
```

## Integration with Development Tools

### GitHub Integration

- Use Linear's suggested git branch names
- Reference Linear issues in commit messages: "CET-123: Add feature"
- PRs automatically link to Linear issues
- PR status updates Linear issue status
- Merge events can auto-complete issues

### CI/CD Integration

- Post test results as comments on Linear issues
- Update issues on deploy events
- Link deployment URLs in issue comments
- Track deployment status per issue

### Slack Integration

- Get notifications for assigned issues
- Update Linear from Slack
- Share issue links in team channels
- Daily standup reminders with your issues

## Common Workflows for Application Types

### Frontend Application Development

```workflow
1. Design → Development workflow:
   - Create issue from design handoff
   - labels: ["frontend", "ui", "design-ready"]
   - Link to Figma/design files
   - Break into component-level issues

2. Component development:
   - Parent: "User Profile Page"
     ├─ "ProfileHeader component"
     ├─ "ProfileStats component"
     ├─ "ProfileSettings component"
     └─ "Profile routing and layout"

3. Integration and testing:
   - Create issues for:
     - Integration testing
     - Accessibility audit
     - Performance optimization
     - Browser compatibility testing
```

### Full-Stack Application Development

```workflow
1. Feature planning:
   - Parent issue: "Payment Processing Feature"
   - Children:
     - Database schema design
     - API endpoint implementation
     - Frontend payment form
     - Payment provider integration
     - Error handling and logging
     - Testing (unit, integration, E2E)

2. Parallel development:
   - Backend team: API and database
   - Frontend team: UI components (mocked API)
   - Coordinate through Linear comments
   - Track blockers and dependencies

3. Integration phase:
   - Create integration testing issue
   - Link all related issues
   - Coordinate deployment
```

### Mobile Application Development

```workflow
1. Platform-specific tracking:
   - labels: ["ios", "android", "shared"]
   - Track platform-specific bugs separately
   - Share code issues labeled "shared"

2. Release management:
   - Project per release: "iOS v2.1.0"
   - Issues per feature/fix in release
   - Track App Store/Play Store submissions
   - Link to release notes

3. Testing workflow:
   - Device testing: Track per device type
   - OS version compatibility
   - Performance on different hardware
   - Beta testing feedback
```

## Troubleshooting

### Common Issues

**Issue not appearing in filters**
- Check if archived: Set includeArchived: true
- Verify team name/ID is correct
- Check status matches expected state

**Cannot create issue**
- Ensure team parameter is provided (required)
- Verify label names exist if using labels
- Check project name/ID is valid
- Ensure assignee exists in workspace

**Cannot update issue**
- Verify issue ID is correct (use get_issue first)
- Check you have permission to modify issue
- Ensure state name is valid (use list_issue_statuses)
- Verify label names exist

**Projects not loading**
- Check includeArchived setting
- Verify team filter is correct
- Try without filters to see all projects

## Advanced Usage

### Automated Project Setup

Create a complete project structure programmatically:

```example
1. Create project
2. Create label set for project
3. Create epic issues (parent issues)
4. Create sub-issues for each epic
5. Assign issues to team members
6. Organize into first cycle
7. Generate project documentation
```

### Metrics and Reporting

Track development metrics:

```example
// Velocity tracking
- Issues completed per cycle
- Average time to complete
- Blockers frequency

// Quality metrics
- Bug rate per feature
- Time to fix bugs
- Reopened issues

// Team metrics
- Issues per team member
- Workload balance
- Completion rates
```

### Custom Workflows

Design workflows matching your process:

```example
// Example: Feature flag workflow
States:
1. Backlog (unstarted)
2. Ready for Dev (unstarted)
3. In Development (started)
4. In Review (started)
5. In QA (started)
6. Deployed with Flag (started)
7. Flag Enabled (started)
8. Monitoring (started)
9. Flag Removed (completed)
10. Done (completed)
```

## Quick Reference

### Essential Shortcuts

```
My active work:
  assignee: "me", state: "In Progress"

Team backlog:
  team: "{team-name}", state: "Backlog", orderBy: "updatedAt"

High priority issues:
  priority: 2, state: "Todo", orderBy: "createdAt"

Recently updated:
  orderBy: "updatedAt", limit: 20

Bugs needing triage:
  label: "bug", state: "Backlog", orderBy: "createdAt"
```

### Issue Creation Template

```
title: "[TYPE] Clear, actionable title"
description: |
  ## Overview
  {What and why}

  ## Tasks
  - [ ] Task 1
  - [ ] Task 2

  ## Acceptance Criteria
  - Criterion 1
  - Criterion 2
team: "{team-name}"
assignee: "me"
labels: ["{type}", "{area}", "{priority}"]
project: "{project-name}"
priority: 3
```

## Resources

- Linear Documentation: https://linear.app/docs
- Linear API Reference: https://developers.linear.app/docs
- Linear Blog: https://linear.app/blog
- GitHub Integration Guide: https://linear.app/docs/github
- Linear Keyboard Shortcuts: linear.app/shortcuts

---

**Skill Version**: 1.0.0
**Last Updated**: October 2025
**Skill Category**: Project Management, Development Acceleration, Team Collaboration
**Compatible With**: Linear MCP Server, GitHub, CI/CD pipelines
