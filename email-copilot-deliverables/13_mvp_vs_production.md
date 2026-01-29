# MVP מול Production

## MVP
- Telegram chat
- Gmail: search/get/thread/summarize/label/archive/mark read/create draft
- Confirmation gate + Dry-run לבאלק
- Strict JSON contract + validation
- Idempotency בסיסי + audit logs
- תיעוד 10 תרחישי בדיקה

## Production (הרחבות)
- UI ווב עם approve actions ו-history
- Postgres מלא + Redis/queues
- RBAC + multi inbox
- Vector store לחיפוש סמנטי על תקצירים
- Observability (metrics/alerts/tracing)
- Policy engine מתקדם (newsletter/VIP/שעות עבודה)
- טיפול מצורפים והעלאת קבצים
