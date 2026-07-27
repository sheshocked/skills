---
name: landing-page-conversion
description: 
category: design
tags: [landing-page-conversion]
---

## When to Use
Building high-converting landing pages for products, campaigns, signups, or launches. When a single page must communicate value and drive a specific action.

## Core Concepts
- **Conversion funnel**: Awareness → Interest → Desire → Action (AIDA framework)
- **Value proposition**: Clear statement of what you do, who it's for, and why it's better — in < 5 seconds
- **Social proof**: Testimonials, logos, case studies, usage numbers reduce perceived risk
- **Visual hierarchy**: Hero → benefit statements → proof → CTA → details
- **Single CTA focus**: One primary action per page; secondary CTAs should be visually subordinate
- **Above the fold**: Hero section must communicate value without scrolling
- **Urgency/scarcity**: Time-limited offers, limited spots — use ethically and truthfully

## Workflow
1. Define the single conversion goal (signup, purchase, download, demo request)
2. Write the value proposition: [Audience] gets [outcome] by [mechanism] unlike [alternative]
3. Design hero section: headline (value prop), subheadline, primary CTA, hero image/video
4. Build benefit sections: 3-4 key benefits with icons, short copy, and supporting visuals
5. Add social proof: customer logos, testimonials with photos, usage statistics
6. Address objections: FAQ section or comparison table
7. Repeat CTA after key sections — users need multiple touchpoints
8. A/B test headlines, CTAs, hero images — small changes compound

## Key Patterns
- **Hero section structure**:
  ```
  Headline: 6-10 words, states the outcome
  Subheadline: 1-2 sentences explaining how
  CTA button: Action-oriented ("Start building", not "Submit")
  Hero image: Product in context or happy user
  ```
- **Social proof bar**: Row of 5-8 customer logos below hero — builds instant credibility
- **Before/After comparison**: Show the pain (before) and the solution (after) side by side
- **Pricing anchor**: Show annual price with monthly equivalent — "Save 20% with annual ($15/mo vs $19/mo)"
- **Exit-intent popup**: Trigger on mouse-leave for desktop — offer discount or content upgrade
- **Sticky CTA bar** (mobile): Fixed bottom bar with CTA button — always accessible on scroll

## Pitfalls
- Multiple competing CTAs — "Sign up" + "Learn more" + "Watch demo" = user paralysis
- Generic hero copy — "Welcome to our platform" wastes precious above-the-fold space
- No mobile optimization — 60%+ traffic is mobile; sticky CTA and thumb-friendly buttons required
- Slow load time — every 1s delay reduces conversion by ~7%; optimize images, lazy-load non-critical
- Missing trust signals — no testimonials, no logos, no security badges near forms
- Overusing urgency — countdown timers that reset on reload destroy trust

## Verification
- 5-second test: show page for 5s, ask "what does this company do and should you care?" — users answer correctly
- Heatmap analysis: are users clicking the primary CTA? Where do they scroll to?
- Form completion rate: < 70%? Simplify fields (name + email only for top-of-funnel)
- Load time: LCP < 2.5s, FID < 100ms, CLS < 0.1
- A/B test: always be running at least one experiment on headlines or CTA copy