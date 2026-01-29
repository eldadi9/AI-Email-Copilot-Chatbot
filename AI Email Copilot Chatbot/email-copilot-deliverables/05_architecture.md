# ארכיטקטורה מוצעת End-to-End

## רכיבים
- Email provider: Gmail OAuth
- UI: Telegram או Webhook
- n8n: orchestration + safety + execution + logging
- AI Agent: מודל שמחזיר JSON קשיח (decision contract)
- Storage: n8n Data Store (MVP) או Postgres (Production)
- Vector store: לא חובה ב-MVP; אופציונלי לפרודקשן לחיפוש סמנטי
- Logging/Monitoring: n8n execution logs + audit table (Data Store/DB)

## חלוקת אחריות

### Agent (Decision)
- intent detection, tool selection, פרמטרים
- סיכום/סיווג/דחיפות
- ניסוח טיוטות
- קביעה needs_confirmation
- לא טוען שביצע פעולה בפועל

### n8n (Execution + Safety)
- ולידציה לסכמות, enforce confirmation gate
- dry-run בבאלק
- ביצוע Gmail actions
- retries, rate limits, idempotency
- audit logging והודעות תוצאה בפועל
