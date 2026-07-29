---
name: premium-animations-motion
description: Structure physics-backed spring animation easing curves and smooth micro-interactions in Framer Motion.
category: design
tags: [animations, framer-motion, motion-design, web-design, micro-interactions]
---

# Premium Spring Animations & Motion Masterclass

## When to Use
Use to build high-end UI micro-interactions (e.g. active toggle transitions, status changes, scroll effects) inspired by premium applications.

## Prerequisites
- Framer Motion library installed (`npm install framer-motion`).

## Workflow
1. Select physics variables (stiffness, damping, mass) instead of linear time-based transitions.
2. Build interactive animations triggers.
3. Configure layout transitions to prevent layout shifts.

## Key Patterns

### React Connection Toggle (ToggleButton.jsx)
```jsx
import { motion } from "framer-motion";
import { useState } from "react";

export function ToggleButton({ isConnected, onToggle }) {
  // Premium spring physics properties
  const springConfig = { type: "spring", stiffness: 500, damping: 30, mass: 0.8 };

  return (
    <div
      onClick={onToggle}
      className={`w-16 h-10 flex items-center rounded-full p-1 cursor-pointer transition-colors duration-300 ${
        isConnected ? "bg-blue-600" : "bg-gray-800"
      }`}
    >
      <motion.div
        layout
        transition={springConfig}
        className="w-8 h-8 rounded-full bg-white shadow-md"
        style={{
          x: isConnected ? "24px" : "0px"
        }}
      />
    </div>
  );
}
```

## Pitfalls
- **Linear duration easing:** Linear timing configurations (`duration: 0.3s`) feel artificial and mechanical. Always utilize physical spring metrics.
- **Re-triggering layout shifts:** Layout changes during animating states trigger browser repaints. Ensure bounds are fixed.

## Verification
- Test rendering loop performance at 120Hz refresh rates.
- Verify animation state transitions match user inputs dynamically.
