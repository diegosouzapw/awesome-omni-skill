---
name: action-creator
description: Creates user-specific one-click action templates that execute email operations when clicked in the chat interface. Use when user wants reusable actions for their specific workflows (send payment reminder to ACME Corp, forward bugs to engineering, archive old newsletters from specific sources).
allowed-tools: Write, Edit, Read, Glob
---

# Action Creator

Creates TypeScript action template files that define reusable, user-specific operations users can execute with one click in the chat interface.

## When to Use This Skill

Use this skill when the user wants to:
- Create reusable actions for their specific workflows ("I often need to send payment reminders to ACME Corp")
- Set up one-click operations for their vendors/customers ("Forward bugs to engineering team")
- Automate repetitive email tasks with their specific context ("Archive newsletters from TechCrunch/Morning Brew")
- Build personalized email management tools for their business processes

**Key difference from listeners**: Actions are **user-triggered** (clicked in chat), while listeners are **event-triggered** (automatic).

## How Actions Work

Actions are TypeScript files in `agent/custom_scripts/actions/` that:
1. Export a `config` object defining the template metadata and parameter schema
2. Export a `handler` function that executes the operation with given parameters
3. Use `ActionContext` methods to perform operations (email API, send emails, call AI, etc.)

The agent creates **action instances** during conversation by providing specific parameters to these templates, which appear as clickable buttons in the chat.

## Creating an Action Template

### 1. Understand User-Specific Workflow

Parse the user's request to identify:
- **User context**: Who are their specific vendors/customers/teams?
- **Operation**: What specific action do they need? (send to ACME Corp, forward to engineering team, etc.)
- **Parameters**: What varies per execution? (invoice number, priority level, days old)
- **Frequency**: How often will they use this?

### 2. Write the Action Template File

Create a file in `agent/custom_scripts/actions/` with this structure:

```typescript
import type { ActionTemplate, ActionContext, ActionResult } from "../types";

export const config: ActionTemplate = {
  id: "unique_action_id",                    // kebab-case, user-specific
  name: "Human Readable Name",                // For UI display
  description: "What this action does",       // Explain the operation
  icon: "📨",                                 // Optional emoji icon
  parameterSchema: {
    type: "object",
    properties: {
      paramName: {
        type: "string",                      // or "number", "boolean"
        description: "Parameter description",
        enum: ["option1", "option2"],        // Optional: restrict values
        default: "defaultValue"              // Optional: default value
      }
    },
    required: ["paramName"]                  // List required parameters
  }
};

export async function handler(
  params: Record<string, any>,
  context: ActionContext
): Promise<ActionResult> {
  const { paramName } = params;

  context.log(`Starting action: ${config.name}`);

  try {
    // 1. Perform operations using context methods
    // 2. Use AI for intelligent processing if needed
    // 3. Update emails, send emails, etc.

    context.notify("Action completed successfully", {
      type: "success",
      priority: "normal"
    });

    return {
      success: true,
      message: "Action completed successfully",
      data: { /* optional structured data */ },
      refreshInbox: true  // Optional: refresh inbox after action
    };
  } catch (error: any) {
    context.log(`Action failed: ${error}`, "error");
    return {
      success: false,
      message: `Failed: ${error.message}`
    };
  }
}
```

### 3. File Naming Convention

Use kebab-case that reflects the **user-specific** operation:
- `send-payment-reminder-to-acme.ts` (not `send-email.ts`)
- `forward-bugs-to-engineering.ts` (not `forward-email.ts`)
- `archive-newsletters-from-techcrunch.ts` (not `archive-emails.ts`)
- `summarize-weekly-updates-from-ceo.ts` (not `summarize-emails.ts`)

**Important**: Templates should be specific to the user's actual workflows, vendors, teams, and processes.

### 4. Available Context Methods

The `ActionContext` provides these capabilities:

```typescript
// Email API operations
const emails = await context.emailAPI.getInbox({ limit: 30, includeRead: false });
const results = await context.emailAPI.searchEmails({ from: "sender@example.com" });
const results = await context.emailAPI.searchWithGmailQuery("from:sender after:2024/01/01");
const emails = await context.emailAPI.getEmailsByIds(["id1", "id2"]);
const email = await context.emailAPI.getEmailById("email-id");

// Direct email operations
await context.archiveEmail(emailId);
await context.starEmail(emailId);
await context.unstarEmail(emailId);
await context.markAsRead(emailId);
await context.markAsUnread(emailId);
await context.addLabel(emailId, "label-name");
await context.removeLabel(emailId, "label-name");

// Send emails
const result = await context.sendEmail({
  to: "recipient@example.com",
  subject: "Email subject",
  body: "Email body content",
  cc: "cc@example.com",           // Optional
  bcc: "bcc@example.com",          // Optional
  replyTo: "reply@example.com"     // Optional
});

// AI-powered processing
const analysis = await context.callAgent<ResultType>({
  prompt: "Analyze this email and extract key info...",
  systemPrompt: "You are an expert at...",  // Optional
  tools: ["Read", "WebSearch"],              // Optional
  maxTokens: 2000                            // Optional
});

// Session messaging (inject into chat)
context.addUserMessage("User said this");
context.addAssistantMessage("Assistant responds");
context.addSystemMessage("System notification");

// Notifications
context.notify("Operation completed", {
  priority: "high" | "normal" | "low",
  type: "info" | "success" | "warning" | "error"
});

// External API access
const response = await context.fetch("https://api.example.com/data");
const data = await response.json();

// Logging (visible in server logs)
context.log("Info message", "info");
context.log("Warning message", "warn");
context.log("Error message", "error");
```

## Action Result

Always return an `ActionResult` object:

```typescript
return {
  success: true,                          // Required: boolean
  message: "Human-readable result",       // Required: string
  data: { key: "value" },                // Optional: structured data
  suggestedActions: [],                   // Optional: follow-up actions
  refreshInbox: true                      // Optional: refresh inbox
};
```

## Examples and Templates

Reference the template files for common patterns:

- **[send-payment-reminder.ts](templates/send-payment-reminder.ts)**: Send invoice reminders to specific vendor
- **[forward-bug-report.ts](templates/forward-bug-report.ts)**: Forward bugs to engineering with AI analysis
- **[archive-old-newsletters.ts](templates/archive-old-newsletters.ts)**: Archive newsletters from specific sources

## Best Practices

1. **User-Specific Templates**: Create templates tailored to user's actual vendors, customers, teams, and processes
2. **Descriptive Naming**: Use specific names that reflect the operation (not generic like "send-email")
3. **Rich Parameter Schemas**: Define clear parameter types with descriptions
4. **AI-Powered**: Use `context.callAgent()` for intelligent processing
5. **Error Handling**: Always wrap operations in try-catch and return meaningful errors
6. **Clear Messages**: Return human-readable success/failure messages
7. **Idempotency**: Design handlers to be safely re-runnable when possible
8. **Logging**: Use `context.log()` for debugging and audit trail

## Parameter Schema Guidelines

Define parameters using JSON Schema:

```typescript
parameterSchema: {
  type: "object",
  properties: {
    // String parameter
    emailId: {
      type: "string",
      description: "Email ID to process"
    },

    // Number parameter with default
    daysOld: {
      type: "number",
      description: "Number of days old",
      default: 30
    },

    // Enum parameter (dropdown)
    priority: {
      type: "string",
      description: "Priority level",
      enum: ["P0 - Critical", "P1 - High", "P2 - Medium", "P3 - Low"]
    },

    // Boolean parameter
    sendNotification: {
      type: "boolean",
      description: "Send notification when complete"
    }
  },
  required: ["emailId", "priority"]  // List required params
}
```

## Creating the File

When the user requests an action template:

1. **Clarify user-specific context**:
   - Who are their vendors/customers/teams?
   - What are their specific workflows?
   - What parameters vary per execution?

2. **Write the TypeScript file** in `agent/custom_scripts/actions/`

3. **Use Write tool** to create the file with:
   - Proper imports from "../types"
   - User-specific config (not generic)
   - Parameter schema with all required fields
   - Handler with error handling
   - Clear success/failure messages

4. **Test parameters**: Ensure all required parameters are defined

5. **Confirm with user** that the action matches their workflow

## Common Patterns

### 1. Send Email to Specific Recipient
User-specific → Compose email with template → Send → Return result

```typescript
const body = `Hi ${recipientName},
Your invoice ${invoiceNumber} for ${amount} is ${daysPastDue} days past due...`;

await context.sendEmail({
  to: "accounts.payable@acmecorp.com",
  subject: `Payment Reminder: Invoice ${invoiceNumber}`,
  body
});
```

### 2. Bulk Email Operation
Search emails → Filter → Apply operation to each → Return count

```typescript
const emails = await context.emailAPI.searchWithGmailQuery(query);
for (const email of emails) {
  await context.archiveEmail(email.messageId);
}
return { success: true, message: `Archived ${emails.length} emails` };
```

### 3. AI-Powered Email Processing
Get email → Call AI to analyze → Use AI result → Take action → Return summary

```typescript
const email = await context.emailAPI.getEmailById(emailId);
const analysis = await context.callAgent({
  prompt: `Analyze this bug report: ${email.body}...`,
  maxTokens: 1000
});
await context.sendEmail({ to: "engineering@company.com", ... });
```

### 4. Email Forwarding with Enhancement
Get email → AI analysis → Compose enhanced forward → Send → Label original

```typescript
const email = await context.emailAPI.getEmailById(emailId);
const analysis = await context.callAgent({ /* analyze */ });
await context.sendEmail({
  to: "team@company.com",
  subject: `[${priority}] ${email.subject}`,
  body: `AI Analysis:\n${analysis}\n\nOriginal:\n${email.body}`
});
await context.addLabel(emailId, "FORWARDED");
```

## Type Imports

Always import types from the correct location:

```typescript
import type { ActionTemplate, ActionContext, ActionResult } from "../types";

// ActionTemplate: Template metadata and parameter schema
// ActionContext: Runtime context with all capabilities
// ActionResult: Return type for handler function
```

## How Users Trigger Actions

After you create an action template:

1. **Agent discovers template**: During conversation, agent reads available actions
2. **Agent creates instance**: Agent provides specific parameters for user's situation
3. **User sees button**: Action instance appears as clickable button in chat
4. **User clicks**: Action executes with pre-filled parameters
5. **Result appears**: Success/failure message shown in chat

Example flow:
```
User: "I need to follow up on the ACME invoice"
Agent: [searches emails, finds Invoice #2024-001 is 15 days overdue]
Agent: Creates action instance with parameters:
  {
    templateId: "send_payment_reminder_acme",
    params: { invoiceNumber: "INV-2024-001", amount: "$5,000", daysPastDue: 15 }
  }
User: [sees button "Send payment reminder to ACME Corp for Invoice #2024-001"]
User: [clicks button]
Action: Executes, sends email, returns "Payment reminder sent to ACME Corp"
```

## Reference

Full specification: See project root `ACTIONS_SPEC.md` for complete details on:
- Complete type definitions
- ActionsManager implementation
- WebSocket protocol
- Frontend integration
- Advanced examples
- Logging and audit trail
