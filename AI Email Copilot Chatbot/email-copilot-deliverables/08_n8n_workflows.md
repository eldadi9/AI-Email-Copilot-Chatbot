# n8n Workflows מפורטים

## WF1: Chat Ingest and Decide
- Trigger: Telegram/Webhook
- Nodes: Trigger -> Load Session -> Build AI Input -> OpenAI Decide -> Validate JSON -> Route intent -> Error handler -> Logging
- Error handling: invalid JSON -> לא לבצע, הודעה "נסח מחדש", לוג

## WF2: Gmail Search Emails
- Trigger: פנימי לפי routing
- Nodes: Gmail Search -> Clean/Redact -> Build context -> return
- Validation: query לא ריק, limit 1-10
- Retries: 429/5xx backoff

## WF3: Summarize Unread
- Nodes: WF2 Search(is:unread) -> OpenAI Summarize batch -> Format -> Respond

## WF4: Get Email by ID
- Nodes: Gmail Get -> Clean/Strip quoted -> Optional Summarize -> Respond

## WF5: Summarize Thread
- Nodes: Gmail Get Thread -> Strip quoted -> OpenAI summarize -> Respond

## WF6: Draft Reply (Preview + Confirm + Create Draft)
- Nodes: Get Email/Thread -> OpenAI draft -> Preview -> Save pending_action
- Confirm handler: create draft -> respond

## WF7: Bulk Newsletter Label + Archive (Dry-run + Confirm)
- Nodes: Search sample + count -> Preview -> Save pending_action
- Confirm: loop apply label + archive עם idempotency -> log -> respond

## WF8: Confirmation Handler
- Trigger: "אשר"/button
- Nodes: Load pending_action -> validate -> execute -> clear -> respond

## WF9: Audit Logging
- Node/step משותף לכל workflows: insert audit record
