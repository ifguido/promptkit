# SendGrid Email Service Integration

## Objective
Implement a comprehensive email service using SendGrid with transactional emails, templates, bulk sending, and email tracking.

## Technology Stack
- Backend: Node.js/Express with TypeScript
- @sendgrid/mail library
- Handlebars or EJS for email templates
- Bull or Bee-Queue for email queue
- Redis for queue management
- Database for email tracking

## Requirements

### 1. Email Types to Support

**Transactional Emails:**
- Welcome email
- Email verification
- Password reset
- Order confirmation
- Receipt/invoice
- Account notifications
- Two-factor authentication codes

**Marketing Emails:**
- Newsletter
- Product announcements
- Promotional campaigns
- Drip campaigns

**System Emails:**
- Error alerts
- Performance reports
- System notifications

### 2. Core Features

**Email Sending:**
- Single email sending
- Bulk email sending (with rate limiting)
- Scheduled emails
- Email with attachments
- HTML and plain text versions
- Dynamic template rendering
- Personalization with variables

**Email Templates:**
- Reusable HTML templates
- Dynamic content insertion
- Responsive email design
- Brand consistency
- Support for SendGrid dynamic templates
- Local template fallback

**Email Queue:**
- Asynchronous email sending
- Retry logic for failures
- Priority queuing
- Rate limiting to respect SendGrid limits
- Job status tracking

**Email Tracking:**
- Delivery status
- Open tracking
- Click tracking
- Bounce handling
- Spam report handling
- Unsubscribe management

### 3. Backend Implementation

**Email Service Class:**
```typescript
class EmailService {
  async sendEmail(options: EmailOptions): Promise<void>;
  async sendTemplatedEmail(templateId: string, data: any): Promise<void>;
  async sendBulkEmails(emails: BulkEmailOptions[]): Promise<void>;
  async scheduleEmail(options: EmailOptions, sendAt: Date): Promise<void>;
  async sendWithAttachment(options: EmailWithAttachment): Promise<void>;
}
```

**API Endpoints:**
```typescript
// Email sending
POST /api/emails/send
POST /api/emails/send-template
POST /api/emails/send-bulk
POST /api/emails/schedule

// Email management
GET /api/emails/:id/status
GET /api/emails/user/:userId
POST /api/emails/:id/resend

// Webhooks
POST /api/webhooks/sendgrid

// Unsubscribe
POST /api/emails/unsubscribe
GET /api/emails/unsubscribe/:token
```

### 4. Email Templates

**Template Structure:**
```
templates/
├── layouts/
│   └── main.hbs
├── emails/
│   ├── welcome.hbs
│   ├── email-verification.hbs
│   ├── password-reset.hbs
│   ├── order-confirmation.hbs
│   ├── receipt.hbs
│   └── newsletter.hbs
└── partials/
    ├── header.hbs
    ├── footer.hbs
    └── button.hbs
```

**Example Template (Handlebars):**
```handlebars
<!DOCTYPE html>
<html>
<head>
  <style>
    /* Responsive email styles */
  </style>
</head>
<body>
  {{> header}}

  <div class="content">
    <h1>Welcome, {{firstName}}!</h1>
    <p>Thank you for joining {{companyName}}.</p>

    {{> button
      text="Verify Your Email"
      url=verificationUrl
    }}
  </div>

  {{> footer
    unsubscribeUrl=unsubscribeUrl
  }}
</body>
</html>
```

### 5. File Structure
```
src/
├── services/
│   ├── email.service.ts
│   ├── template.service.ts
│   └── emailQueue.service.ts
├── controllers/
│   ├── email.controller.ts
│   └── webhook.controller.ts
├── models/
│   ├── EmailLog.model.ts
│   └── Unsubscribe.model.ts
├── templates/
│   ├── layouts/
│   ├── emails/
│   └── partials/
├── queues/
│   └── email.queue.ts
├── webhooks/
│   └── sendgrid.webhook.ts
├── routes/
│   ├── email.routes.ts
│   └── webhook.routes.ts
└── utils/
    ├── emailValidator.ts
    └── emailHelpers.ts
```

### 6. Database Schema

**EmailLog Table:**
```typescript
{
  id: UUID,
  userId: UUID?,
  to: string,
  from: string,
  subject: string,
  templateId: string?,
  messageId: string (SendGrid message ID),
  status: enum (queued, sent, delivered, opened, clicked, bounced, failed),
  sentAt: DateTime?,
  deliveredAt: DateTime?,
  openedAt: DateTime?,
  clickedAt: DateTime?,
  failureReason: string?,
  metadata: JSON,
  createdAt: DateTime,
  updatedAt: DateTime
}
```

**Unsubscribe Table:**
```typescript
{
  id: UUID,
  email: string (unique),
  userId: UUID?,
  reason: string?,
  unsubscribedAt: DateTime,
  createdAt: DateTime
}
```

### 7. Email Service Implementation

```typescript
import sgMail from '@sendgrid/mail';
import handlebars from 'handlebars';
import fs from 'fs/promises';

export class EmailService {
  constructor() {
    sgMail.setApiKey(process.env.SENDGRID_API_KEY);
  }

  async sendEmail(options: EmailOptions): Promise<void> {
    const { to, subject, html, text, from, attachments } = options;

    const msg = {
      to,
      from: from || process.env.SENDGRID_FROM_EMAIL,
      subject,
      html,
      text,
      attachments,
    };

    try {
      const [response] = await sgMail.send(msg);

      await EmailLog.create({
        to,
        from: msg.from,
        subject,
        messageId: response.headers['x-message-id'],
        status: 'sent',
        sentAt: new Date(),
      });
    } catch (error) {
      await EmailLog.create({
        to,
        subject,
        status: 'failed',
        failureReason: error.message,
      });
      throw error;
    }
  }

  async sendTemplatedEmail(
    templateName: string,
    to: string,
    data: any
  ): Promise<void> {
    const html = await this.renderTemplate(templateName, data);
    const text = this.htmlToText(html);

    await this.sendEmail({
      to,
      subject: data.subject,
      html,
      text,
    });
  }

  async renderTemplate(name: string, data: any): Promise<string> {
    const templatePath = `./templates/emails/${name}.hbs`;
    const templateSource = await fs.readFile(templatePath, 'utf-8');
    const template = handlebars.compile(templateSource);
    return template(data);
  }

  async sendBulkEmails(emails: BulkEmailOptions[]): Promise<void> {
    // Add to queue for processing
    for (const email of emails) {
      await emailQueue.add('send-email', email, {
        attempts: 3,
        backoff: {
          type: 'exponential',
          delay: 2000,
        },
      });
    }
  }
}
```

### 8. Email Queue Implementation

```typescript
import Queue from 'bull';

export const emailQueue = new Queue('emails', {
  redis: process.env.REDIS_URL,
});

emailQueue.process('send-email', async (job) => {
  const { to, subject, html, text } = job.data;

  await emailService.sendEmail({ to, subject, html, text });
});

// Rate limiting: max 100 emails per second (SendGrid free tier)
emailQueue.process('send-email', 100, async (job) => {
  // Process with concurrency
});
```

### 9. Webhook Handler

```typescript
export const handleSendGridWebhook = async (req: Request, res: Response) => {
  const events = req.body;

  for (const event of events) {
    const { email, event: eventType, timestamp, sg_message_id } = event;

    await EmailLog.update(
      { messageId: sg_message_id },
      {
        status: mapEventToStatus(eventType),
        [`${eventType}At`]: new Date(timestamp * 1000),
      }
    );

    // Handle specific events
    switch (eventType) {
      case 'bounce':
      case 'dropped':
        await handleBounce(email, event.reason);
        break;
      case 'spamreport':
        await handleSpamReport(email);
        break;
      case 'unsubscribe':
        await handleUnsubscribe(email);
        break;
    }
  }

  res.status(200).json({ received: true });
};
```

### 10. Email Templates Examples

**Welcome Email:**
```typescript
await emailService.sendTemplatedEmail('welcome', user.email, {
  subject: 'Welcome to Our Platform!',
  firstName: user.firstName,
  companyName: 'YourCompany',
  verificationUrl: `${baseUrl}/verify-email?token=${token}`,
  unsubscribeUrl: `${baseUrl}/unsubscribe?token=${unsubToken}`,
});
```

**Password Reset:**
```typescript
await emailService.sendTemplatedEmail('password-reset', user.email, {
  subject: 'Reset Your Password',
  firstName: user.firstName,
  resetUrl: `${baseUrl}/reset-password?token=${resetToken}`,
  expiresIn: '1 hour',
});
```

**Order Confirmation:**
```typescript
await emailService.sendTemplatedEmail('order-confirmation', user.email, {
  subject: `Order Confirmation - #${order.id}`,
  orderNumber: order.id,
  items: order.items,
  total: order.total,
  shippingAddress: order.shippingAddress,
  trackingUrl: order.trackingUrl,
});
```

### 11. Advanced Features

**Personalization:**
```typescript
const personalizations = users.map(user => ({
  to: user.email,
  substitutions: {
    firstName: user.firstName,
    customContent: user.preferences.content,
  },
}));
```

**Scheduled Sending:**
```typescript
await sgMail.send({
  to: 'user@example.com',
  from: 'noreply@yourapp.com',
  subject: 'Scheduled Email',
  html: '<p>This email was scheduled</p>',
  sendAt: Math.floor(Date.now() / 1000) + 3600, // Send in 1 hour
});
```

**A/B Testing:**
```typescript
// Send variant A to 50% of users
// Send variant B to other 50%
// Track which performs better
```

**Unsubscribe Management:**
```typescript
const unsubscribeUrl = generateUnsubscribeUrl(user.id);
// Include in all marketing emails
// Handle unsubscribe requests via webhook or link
```

### 12. Environment Variables
```
SENDGRID_API_KEY=SG.xxx
SENDGRID_FROM_EMAIL=noreply@yourapp.com
SENDGRID_FROM_NAME=Your App Name
SENDGRID_WEBHOOK_URL=https://yourapp.com/api/webhooks/sendgrid
REDIS_URL=redis://localhost:6379
BASE_URL=https://yourapp.com
```

### 13. Testing

**Email Testing Service:**
```typescript
// Use Ethereal for development
import nodemailer from 'nodemailer';

if (process.env.NODE_ENV === 'development') {
  const testAccount = await nodemailer.createTestAccount();
  // Use test account instead of SendGrid
}
```

## Deliverables
- Complete email service with SendGrid integration
- Email queue system for reliable delivery
- Responsive email templates
- Webhook handling for email events
- Email tracking and analytics
- Unsubscribe management
- Comprehensive error handling
- Testing utilities
- Documentation

## Testing Checklist
- [ ] Single email sends successfully
- [ ] Bulk emails are queued and sent
- [ ] Templates render correctly with dynamic data
- [ ] Attachments are included properly
- [ ] Scheduled emails send at correct time
- [ ] Webhooks update email status
- [ ] Bounces are handled correctly
- [ ] Unsubscribes are processed
- [ ] Email queue retries on failure
- [ ] Rate limiting prevents exceeding limits
- [ ] Email logs are created for all sends
- [ ] Error handling provides useful messages
- [ ] Emails are responsive on all devices
- [ ] Plain text version is generated
- [ ] Personalization works correctly
