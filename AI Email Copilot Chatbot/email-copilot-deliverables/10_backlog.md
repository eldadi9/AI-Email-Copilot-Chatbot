# Backlog לפי סדר דף האפיון

## Epic 1: Setup/Infra
- User stories: להרים סביבת n8n עובדת עם Credentials
- Tasks: n8n + DataStore/DB, Telegram/Webhook, OpenAI creds, Gmail OAuth creds
- DoD: Trigger עובד, אפשר להריץ flow בסיסי, credentials נשמרים ב-n8n
- Order: 1
- Risk: Med

## Epic 2: Integrations
- User stories: חיפוש/קריאה/שרשורים/טיוטות/שינוי תוויות
- Tasks: Gmail Search/Get/Thread/Create Draft/Modify
- DoD: כל פעולה עובדת, שגיאות מחזירות הודעה ולוג
- Order: 2
- Risk: Med

## Epic 3: Agent logic
- User stories: Agent מחזיר JSON קשיח, routing בטוח
- Tasks: system prompt, JSON schema validation, confidence rules, confirmation rules
- DoD: invalid JSON נחסם, אין ביצוע ללא ולידציה
- Order: 3
- Risk: High

## Epic 4: Workflows E2E
- Tasks: WF1, WF2, WF3, WF5, WF6, WF7, WF8
- DoD: use cases עובדים מקצה לקצה
- Order: 4
- Risk: High

## Epic 5: Data
- Tasks: session, pending_action, idempotency, audit
- DoD: retries לא עושים כפילויות, יש trace
- Order: 5
- Risk: Med

## Epic 6: Security and Privacy
- Tasks: redaction, strip HTML/quoted, confirmation gate, no body storage
- DoD: אין הדפסת תוכן מלא בלוגים, אין פעולה הרסנית בלי אישור
- Order: 6
- Risk: High

## Epic 7: Testing
- Tasks: לפחות 10 תרחישים כולל invalid JSON, rate limit, HTML heavy, destructive blocked
- DoD: תיעוד תוצאות
- Order: 7
- Risk: Med

## Epic 8: Release and Docs
- Tasks: export workflows, README, תיעוד limitations
- DoD: deliverables קיימים
- Order: 8
- Risk: Low
