# Capabilities

## Chat interface
- Telegram Trigger (מומלץ ל-MVP)
- או Webhook Trigger

## Gmail tools
- Search emails לפי Gmail query syntax עם limit ומיון newest
- Get email by id
- Get thread
- Summarize unread/latest
- Apply/remove label
- Archive
- Mark as read
- Create draft reply (לא לשלוח)

## Safety
- Review לפני פעולה למחיקות/ארכוב/סימון נקרא/תיוג בבאלק/יצירת draft
- Dry-run לבאלק: count + דוגמאות + בקשת אישור

## Reliability
- Idempotency לפי email/thread + action
- Retries עם exponential backoff ל-429 ו-5xx
- fallback כש-AI מחזיר JSON לא תקין: לא לבצע

## Privacy
- מינימיזציה: subject/from/date/snippet
- Redaction אופציונלי
- least privilege scopes + שמירת credentials ב-n8n בלבד
