# Configure Email Notifications

> **Document Type**: How-To Guide (SAMPLE)  
> **Audience**: Backend Developers, DevOps  
> **Difficulty**: Intermediate  
> **Last Updated**: November 5, 2024

---

## Problem

You need to configure email notifications for the Blog Engine using SendGrid as the email service provider.

---

## Prerequisites

- SendGrid account with API key
- Access to environment configuration
- Node.js service running locally

---

## Solution

### Step 1: Install Dependencies

```bash
npm install @sendgrid/mail
```

### Step 2: Configure Environment Variables

Add to `.env` file:

```bash
SENDGRID_API_KEY=SG.your_api_key_here
EMAIL_FROM=noreply@blogengine.com
EMAIL_FROM_NAME=Blog Engine
```

### Step 3: Create Email Service

Create `src/services/email.service.js`:

```javascript
const sgMail = require('@sendgrid/mail');

sgMail.setApiKey(process.env.SENDGRID_API_KEY);

class EmailService {
  async sendWelcomeEmail(user) {
    const msg = {
      to: user.email,
      from: {
        email: process.env.EMAIL_FROM,
        name: process.env.EMAIL_FROM_NAME
      },
      subject: 'Welcome to Blog Engine!',
      text: `Hi ${user.name}, welcome to Blog Engine!`,
      html: `<h1>Welcome ${user.name}!</h1><p>Thanks for joining Blog Engine.</p>`
    };
    
    await sgMail.send(msg);
  }

  async sendPostPublishedNotification(post, subscribers) {
    const emails = subscribers.map(sub => ({
      to: sub.email,
      from: process.env.EMAIL_FROM,
      subject: `New post: ${post.title}`,
      html: `<p>A new post has been published: <strong>${post.title}</strong></p>`
    }));
    
    await sgMail.send(emails);
  }
}

module.exports = new EmailService();
```

### Step 4: Use in Your Application

```javascript
const emailService = require('./services/email.service');

// In user registration handler
router.post('/api/auth/register', async (req, res) => {
  const user = await UserService.create(req.body);
  
  // Send welcome email
  await emailService.sendWelcomeEmail(user);
  
  res.status(201).json(user);
});
```

### Step 5: Test Email Sending

```bash
# Create test script
node scripts/test-email.js
```

`scripts/test-email.js`:
```javascript
const emailService = require('../src/services/email.service');

const testUser = {
  name: 'Test User',
  email: 'test@example.com'
};

emailService.sendWelcomeEmail(testUser)
  .then(() => console.log('Email sent successfully!'))
  .catch(err => console.error('Email failed:', err));
```

---

## Verification

1. Check SendGrid dashboard for sent emails
2. Check application logs for success/errors
3. Verify email received in inbox (check spam folder)

---

## Troubleshooting

**Email not sending:**
- Verify API key is correct
- Check SendGrid account status
- Review application logs for errors

**Email going to spam:**
- Configure SPF and DKIM records
- Add sender verification in SendGrid
- Use authenticated domain

---

## Related

- [Notification Service Design](../../docs/architecture/notification-service.md)
- [SendGrid Documentation](https://docs.sendgrid.com)
- [Environment Variables Reference](../../reference/configuration/parameters.md)
