---
name: notification-design-eng
description: 
category: general
tags: [notification-design-eng]
---

## When to Use
Design notification systems: email/SMS/push templates, delivery, unsubscribe compliance.

## Email Template
```html
<!DOCTYPE html>
<html>
<head>
    <style>
        body { font-family: Arial; max-width: 600px; margin: 0 auto; }
        .header { background: #1a1a2e; color: white; padding: 20px; }
        .content { padding: 20px; }
        .button { background: #16a085; color: white; padding: 12px 24px; text-decoration: none; border-radius: 4px; }
    </style>
</head>
<body>
    <div class="header"><h1>SurfShield</h1></div>
    <div class="content">
        <h2>{{title}}</h2>
        <p>{{message}}</p>
        <a href="{{cta_url}}" class="button">{{cta_text}}</a>
    </div>
    <div style="padding: 20px; font-size: 12px; color: #666;">
        <p><a href="{{unsubscribe_url}}">Unsubscribe</a></p>
    </div>
</body>
</html>
```

## Pitfalls
- **CAN-SPAM/GDPR**: Must include unsubscribe link
- **Rate limiting**: Don't send too many notifications
- **Templates**: Use transactional email services (SendGrid, Resend)
- **Deliverability**: Set up SPF/DKIM/DMARC

## Verification
- Test email rendering across clients
- Check unsubscribe flow works
- Verify SPF/DKIM records