# ניצול חיבור ChatGPT ל-n8n כאפליקציה

## מה נכון לעשות בתוך n8n
- ביצוע אמיתי: fetch, modify, draft creation
- ולידציה, confirmation gates, dry-run
- retries, rate limit handling, idempotency
- audit logging ותשובת תוצאה בפועל

## מה נכון להשאיר ל-Agent
- intent detection, הצעת query, סיכומים, ניסוח טיוטות
- החלטה needs_confirmation
- לא לבצע פעולות בפועל

## Interface ברור של tools
הגדרה: כל tool = workflow עם contract קשיח (Webhook פנימי או קריאה פנימית).

שמות מומלצים:
- gmail.search (query, limit, sort) -> results[]
- gmail.get (email_id) -> email meta + cleaned snippet/body
- gmail.thread (thread_id) -> messages[]
- gmail.draft (thread_id/email_id, draft_text) -> draft_id
- gmail.label (email_ids/thread_id, add/remove) -> applied_count
- gmail.archive (email_ids/thread_id) -> applied_count
- gmail.mark_read (email_ids/thread_id) -> applied_count

כל tool מחזיר גם:
- execution_confirmed: true/false
- errors: []

כך ה-Agent לא "מדמיין" ביצוע.
