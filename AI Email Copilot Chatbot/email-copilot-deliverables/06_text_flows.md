# תרשימי זרימה טקסטואליים

## Flow A: הודעת צ'אט כללית
1. Trigger Telegram/Webhook
2. Load session (DataStore/DB)
3. Build minimal context
4. OpenAI Decide Intent (JSON)
5. Validate JSON schema
6. אם ask_clarifying -> שאלת הבהרה -> שמירת session
7. אחרת validate params
8. אם needs_confirmation -> preview + בקשת אישור + שמירת pending_action
9. אחרת execute Gmail action
10. Audit log + response

## Flow B: אישור פעולה
1. Trigger "אשר"/button
2. Load pending_action
3. Re-validate + idempotency
4. Execute action(s)
5. Log + clear pending_action
6. Send result summary
