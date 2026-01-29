# README

## מטרת המערכת
AI Email Copilot Chatbot: צטבוט שמסייע לחפש, לסכם, לתייג, לארכב, לסמן נקרא, ולהכין טיוטות תשובה ב-Gmail עם שכבת בטיחות ואישור.

## הרמה מקומית (מינימום)
1. הרם n8n (Docker מומלץ) + Data Store או Postgres
2. צור Telegram bot או הגדר Webhook
3. הגדר Credentials: OpenAI, Gmail OAuth ב-n8n
4. ייבא את ה-workflows והפעל

## env variables (דוגמה)
- N8N_HOST
- N8N_PORT
- N8N_ENCRYPTION_KEY
- TELEGRAM_BOT_TOKEN (אם Telegram)
- OPENAI_API_KEY (ב-Credentials של n8n)
- GMAIL_OAUTH_CLIENT_ID, GMAIL_OAUTH_CLIENT_SECRET (ב-Credentials של n8n)
- DATASTORE_BACKEND (datastore או postgres)

## חיבור Gmail
- צור OAuth app בגוגל
- שמור אסימונים רק ב-n8n Credentials
- השתמש ב-scopes מינימליים בהתאם לפעולות

## בדיקות
- הרץ 10 תרחישים ידניים לפחות ותעד execution logs

## הוספת Use Case חדש
1. הוסף intent חדש לחוזה ה-Agent
2. הוסף routing ב-WF1
3. הוסף/עדכן workflow ביצוע עם ולידציה + confirmation
4. הוסף תרחיש בדיקה ותיעוד
