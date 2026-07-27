---
name: ai-coding-workflows
description: 
category: ai-tools
tags: [ai-coding-workflows]
---

## When to Use
Use AI effectively for coding: context management, diff discipline, verification patterns.

## Best Practices
1. **Context**: Provide minimal but sufficient context
2. **Verification**: Always review AI-generated code
3. **Iteration**: Refine prompts based on output
4. **Testing**: Test AI code as rigorously as human code

## Prompt Patterns
```
// Specific task
"Write a Python function that reads a CSV, filters rows where column X > threshold, and returns the result as a pandas DataFrame. Include type hints and docstring."

// Code review
"Review this function for security issues, performance problems, and edge cases: [code]"

// Refactoring
"Refactor this function to be more readable and add error handling: [code]"
```

## Pitfalls
- **Blind trust**: AI code may have subtle bugs
- **Context overflow**: Too much code confuses the model
- **Security**: AI may introduce vulnerabilities
- **Dependencies**: Verify suggested packages exist and are maintained

## Verification
- Run tests on AI-generated code
- Check for security vulnerabilities
- Verify edge cases are handled
- Review imports and dependencies