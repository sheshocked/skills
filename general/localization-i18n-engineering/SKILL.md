---
name: localization-i18n-engineering
description: 
category: general
tags: [localization-i18n-engineering]
---

## When to Use
Internationalize software: i18n frameworks, plural rules, date/currency formatting, RTL support.

## React i18n (react-intl)
```javascript
import { FormattedMessage, FormattedNumber } from 'react-intl';

// Translation
<FormattedMessage id="greeting" values={{ name: "Ali" }} />

// Number formatting
<FormattedNumber value={1234567} style="currency" currency="USD" />

// Date formatting
<FormattedDate value={new Date()} year="numeric" month="long" day="numeric" />
```

## Message Format
```json
{
    "greeting": "Hello, {name}!",
    "items": "{count, plural, =0 {No items} one {# item} other {# items}}"
}
```

## Pitfalls
- **Plural rules**: Different languages have different plural forms
- **Date formats**: MM/DD vs DD/MM vs YYYY-MM-DD
- **Currency**: Position of currency symbol varies
- **RTL**: Mirror entire layout, not just text

## Verification
- Test with RTL language (Arabic/Persian)
- Verify plural forms
- Check date/currency formatting