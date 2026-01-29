# Use Cases (לפי המסמך)

## UC1: סיכום מיילים לא נקראו אחרונים
- Trigger: "Summarize my latest unread emails" / "מה דחוף היום"
- Input: chat_text, user_id, limit אופציונלי
- Output: תקצירים קצרים + urgency 1-5 + קטגוריה + next action
- Success: סיכום שימושי בלי חשיפת תוכן מלא, בגבולות limit
- Edge: אין מיילים לא נקראו, HTML כבד, מייל קצר מאד

## UC2: חיפוש מיילים לפי שאילתה
- Trigger: "Find emails from X about invoice"
- Input: query, limit (ברירת מחדל 5-10)
- Output: תוצאות (id, subject, from, date, snippet) + המלצה להמשך
- Success: query תקין, מיון newest, limit פעיל
- Edge: query אמביגואי, יותר מדי תוצאות, rate limit

## UC3: קריאת מייל לפי id
- Trigger: בחירה מתוצאה או "read email id ..."
- Input: email_id
- Output: תקציר + דחיפות + next action (+ snippet נקי)
- Success: בלי HTML גולמי אם לא צריך, redaction אם הופעל
- Edge: id לא קיים/אין הרשאה, body ריק/רק מצורפים

## UC4: סיכום שרשור (Thread)
- Trigger: "Summarize this thread"
- Input: thread_id או email_id
- Output: תקציר + action items + urgency + next action
- Success: להסיר quoted history כשאפשר, לא להמציא
- Edge: שרשור ארוך מאד, שפה מעורבת, ניוזלטרים

## UC5: יצירת טיוטת תשובה (Draft)
- Trigger: "Draft a reply..."
- Input: מייל אחרון בשרשור + הוראות המשתמש
- Output: טיוטה + preview למשתמש
- Success: never send, תמיד preview ואישור לפני יצירה
- Edge: חסר מידע קריטי, טון לא מתאים, JSON לא תקין

## UC6: תיוג וארכוב ניוזלטרים בבאלק עם אישור
- Trigger: "Label all newsletters... and archive"
- Input: query, label, limit/batch
- Output: Dry-run + אישור, ואז ביצוע + סיכום
- Success: אישור חובה, idempotency, audit
- Edge: זיהוי לא ודאי, label לא קיים, retries

## UC7: סימון נקרא/ארכוב/שינוי תוויות
- Trigger: "mark read", "archive", "apply/remove label"
- Input: email_id/ids/query
- Output: preview בבאלק + אישור + ביצוע
- Success: חסימה ללא אישור לפעולה הרסנית
- Edge: ניסיון בלי אישור, חסר limit

## UC8: התאוששות מ-JSON לא תקין
- Trigger: AI מחזיר פלט לא עומד בסכמה
- Output: הודעה "נסח מחדש" + לוג, בלי פעולות Gmail
- Success: 0 ביצוע

## UC9: טיפול ב-rate limit ו-retries
- Trigger: Gmail 429/5xx
- Output: retry + backoff, ואם נכשל הודעה + לוג
- Success: no duplicates באמצעות idempotency
