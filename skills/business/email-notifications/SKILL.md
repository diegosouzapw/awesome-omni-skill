---
name: email-notifications
description: Email template and notification patterns using Resend
alwaysApply: false
applyTo:
  - "services/notification-service/**/*.ts"
  - "services/notification-service/templates/**"
---

# Email Notifications Skill

Guide for email templates and notifications in Splits Network.

## Purpose

- **Email Templates**: Professional HTML templates with branding
- **Resend Integration**: Sending transactional emails
- **Event-Driven**: Email triggered by domain events
- **Template Variables**: Dynamic content in emails

## When to Use

- Creating new email templates
- Sending transactional emails
- Processing domain events for notifications
- Testing email delivery

## Core Patterns

### 1. Email Template Structure

```typescript
export function applicationCreatedEmail(data: {
  candidateName: string;
  jobTitle: string;
  companyName: string;
  applicationUrl: string;
}) {
  return {
    subject: `Application Received: ${data.jobTitle}`,
    html: `
      <!DOCTYPE html>
      <html>
        <head>
          <meta charset="utf-8">
          <meta name="viewport" content="width=device-width, initial-scale=1.0">
          <style>
            body { font-family: Arial, sans-serif; line-height: 1.6; }
            .container { max-width: 600px; margin: 0 auto; padding: 20px; }
            .button { background: #0066cc; color: white; padding: 12px 24px; text-decoration: none; }
          </style>
        </head>
        <body>
          <div class="container">
            <h1>Application Received</h1>
            <p>Hi ${data.candidateName},</p>
            <p>We've received your application for <strong>${data.jobTitle}</strong> at ${data.companyName}.</p>
            <p>We'll review your application and get back to you soon.</p>
            <a href="${data.applicationUrl}" class="button">View Application</a>
          </div>
        </body>
      </html>
    `
  };
}
```

### 2. Resend Integration

```typescript
import { Resend } from 'resend';

const resend = new Resend(process.env.RESEND_API_KEY);

export async function sendEmail(
  to: string,
  subject: string,
  html: string
) {
  try {
    const { data, error } = await resend.emails.send({
      from: 'Splits Network <notifications@splits.network>',
      to,
      subject,
      html
    });
    
    if (error) {
      console.error('Failed to send email:', error);
      throw error;
    }
    
    return data;
  } catch (error) {
    console.error('Email error:', error);
    throw error;
  }
}
```

### 3. Event-Driven Email

```typescript
// Consumer listens for events
export class ApplicationEventsConsumer {
  async handleApplicationCreated(payload: any) {
    const { applicationId, candidateId, jobId } = payload;
    
    // Fetch data for email
    const application = await this.fetchApplication(applicationId);
    const candidate = await this.fetchCandidate(candidateId);
    const job = await this.fetchJob(jobId);
    
    // Generate email
    const email = applicationCreatedEmail({
      candidateName: candidate.name,
      jobTitle: job.title,
      companyName: job.company.name,
      applicationUrl: `${process.env.APP_URL}/applications/${applicationId}`
    });
    
    // Send email
    await sendEmail(candidate.email, email.subject, email.html);
  }
}
```

### 4. Email Types

```typescript
// Application emails
applicationCreatedEmail()
applicationStageChangedEmail()
applicationAcceptedEmail()

// Placement emails
placementCreatedEmail()
placementActivatedEmail()

// Recruiter emails
recruiterInvitedEmail()
candidateSubmittedEmail()

// Collaboration emails
recruiterAssignedEmail()
teamMemberInvitedEmail()
```

### 5. Template Testing

```typescript
// Generate preview HTML
export function previewEmail() {
  const html = applicationCreatedEmail({
    candidateName: 'John Doe',
    jobTitle: 'Senior Engineer',
    companyName: 'Acme Corp',
    applicationUrl: 'https://app.splits.network/applications/123'
  });
  
  return html;
}

// Test email sending
async function testEmail() {
  await sendEmail(
    'test@example.com',
    'Test Email',
    previewEmail().html
  );
}
```

See [examples/](./examples/) and [references/](./references/).
