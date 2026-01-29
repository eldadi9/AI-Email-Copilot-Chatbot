# עבודה עם Cursor (קצר ומעשי)

## מה לכתוב ב-Cursor
- Gmail client wrapper (אם לא נשענים רק על n8n nodes)
- JSON Schemas + validators
- Docker compose (n8n + postgres)
- utilities: html clean, strip quoted, redaction
- tests (mocks, rate limit, token refresh)

## מה אני מייצר/מבקר
- חוזה Agent <-> n8n
- ארכיטקטורה, guardrails, סדר עבודה, workflows, test plan

## פרומפטים מוכנים ל-Cursor
1. Generate a Node.js Gmail client wrapper with methods: searchMessages, getMessage, getThread, createDraft, modifyLabels, archive, markRead. Include OAuth token refresh, retry with exponential backoff on 429 and 5xx, and TypeScript types.
2. Create a JSON Schema for the AI decision contract: intent, confidence, needs_confirmation, message_to_user, email_insights, action.type, action.params. Add validation examples.
3. Write unit tests for the Gmail wrapper using mocks. Cover rate limit, token refresh, and idempotency key generation.
4. Create Docker compose for n8n + postgres. Include .env.example and minimal security defaults. Document how to import workflows.
5. Generate utility functions: stripQuotedText(emailBody), htmlToTextClean(html), redactPII(text) with basic patterns. Add tests.
