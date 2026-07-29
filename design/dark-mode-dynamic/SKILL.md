---
name: dark-mode-dynamic
description: Establish dynamic theme switching, OLED black styles, and state synchronization across layouts.
category: design
tags: [dark-mode, tailwind-theme, oled-black, theme-sync, css]
---

# Dark Mode Dynamic

## When to Use
Use when designing mobile applications (inspired by premium interfaces like Incy) targeting low-power OLED devices and dark aesthetic preferences.

## Prerequisites
- Tailwind CSS / Jetpack Compose.

## Workflow
1. Define strict Tailwind semantic color configurations (e.g. `bg-primary` matches OLED black `#000000` or navy `#0a0b10`).
2. Sync theme state using local system preferences or user toggle.
3. Render transition animations dynamically.

## Key Patterns
```javascript
// tailwind.config.js snippet
module.exports = {
  darkMode: 'class',
  theme: {
    extend: {
      colors: {
        bg: {
          light: '#ffffff',
          dark: '#000000', // Pure OLED Black
          navy: '#0b0c10'  // Slate Dark
        }
      }
    }
  }
}

// Toggle logic
function toggleTheme() {
  const root = document.documentElement;
  if (root.classList.contains('dark')) {
    root.classList.remove('dark');
    localStorage.theme = 'light';
  } else {
    root.classList.add('dark');
    localStorage.theme = 'dark';
  }
}
```

## Pitfalls
- **Flickering on load:** Ensure theme script runs synchronously before rendering components to prevent white flashbangs.
- **Insufficient text contrast:** Check pure white text against pitch black backgrounds; use soft grays (`#e2e8f0`) to reduce eye strain.

## Verification
- Test rendering under Chrome light/dark simulator.
- Verify user selection persists across reloads.
