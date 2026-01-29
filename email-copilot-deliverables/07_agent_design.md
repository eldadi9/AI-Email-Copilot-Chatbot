# תכנון AI Agent

## תפקידי Agent
- intent detection: search, summarize, read, thread, draft_reply, label, archive, mark_read, ask_clarifying
- tool selection + חילוץ פרמטרים
- guardrails:
  - confidence נמוך -> ask_clarifying
  - needs_confirmation לכל פעולה הרסנית/באלק/draft
  - לא להמציא תוכן מיילים
  - לא לטעון שבוצעה פעולה בלי תוצאת n8n

## Tools interface (שמות מומלצים)
- gmail.search
- gmail.get
- gmail.thread
- gmail.label
- gmail.archive
- gmail.mark_read
- gmail.draft

## Memory
- Session: last_intent, last_email_ids, pending_action, preferences (limit, redaction)
- Idempotency keys: hash(user_id + action + email/thread + params_signature)
- Audit log: timestamp, user, action, params מינימליים, success/failure

## פרטיות
- לא לשמור bodies כברירת מחדל
- redaction לפני שליחה ל-AI (אופציונלי, ברירת מחדל on)
