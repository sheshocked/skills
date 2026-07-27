---
name: chatbot-production
description: 
category: ai-tools
tags: [chatbot-production]
---

## When to Use
Build production chatbots: conversation state, escalation, analytics, multi-channel delivery.

## Architecture
```
User Message → Channel Adapter → Bot Engine → Response
                                  ↓
                            Conversation State
                            Analytics Pipeline
                            Escalation Rules
```

## Key Patterns
```python
class ConversationManager:
    def __init__(self):
        self.sessions = {}  # user_id -> conversation state

    def handle_message(self, user_id, message):
        session = self.sessions.get(user_id, new_session())

        # Check escalation
        if self.should_escalate(session, message):
            return self.escalate_to_human(session)

        # Generate response
        response = self.llm.chat(session.history + [{"role": "user", "content": message}])

        session.history.append({"role": "user", "content": message})
        session.history.append({"role": "assistant", "content": response})

        return response
```

## Pitfalls
- **State management**: Conversation state must be durable
- **Rate limiting**: Prevent abuse
- **Escalation**: Always provide human fallback
- **Privacy**: Don't log sensitive user data

## Verification
- Test conversation flows end-to-end
- Verify escalation works
- Check analytics capture
- Test with concurrent users