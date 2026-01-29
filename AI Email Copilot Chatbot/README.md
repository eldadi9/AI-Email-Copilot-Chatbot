# 🤖 AI Email Copilot Chatbot - Complete Project Documentation

## 📋 Table of Contents
1. [Project Overview](#project-overview)
2. [System Architecture](#system-architecture)
3. [Features & Capabilities](#features--capabilities)
4. [Complete Road Map](#complete-road-map)
5. [Detailed Use Cases](#detailed-use-cases)
6. [AI Agent Specification](#ai-agent-specification)
7. [n8n Workflows](#n8n-workflows)
8. [Data Layer](#data-layer)
9. [Security & Privacy](#security--privacy)
10. [Testing Plan](#testing-plan)
11. [Installation Guide](#installation-guide)
12. [Project Backlog](#project-backlog)
13. [Development Guidelines](#development-guidelines)

---

## 🎯 Project Overview

### What is AI Email Copilot Chatbot?

מערכת "AI Email Copilot Chatbot" היא צ'אטבוט חכם שמדבר עם המשתמש (דרך Telegram או Webhook), ובאישור המשתמש ניגש לתיבת Gmail כדי:
- 🔍 לחפש מיילים
- 📧 לקרוא ולסכם מיילים ושרשורים
- 🏷️ לסווג ולתייג
- ✍️ להכין טיוטות תשובה בתוך Gmail (ללא שליחה אוטומטית)

### Core Principles

**🧠 Agent = Brain (Decision Making)**
- זיהוי כוונות
- חילוץ פרמטרים
- סיווג ודחיפות
- החלטה אם צריך אישור
- **לעולם לא** מבצע פעולות - רק מחזיר החלטות ב-JSON

**⚙️ n8n = Executor (Safe Operations)**
- אימות סכמות ופרמטרים
- Enforcement של confirmation gates
- Dry run לפעולות בכמות
- ביצוע פעולות Gmail
- Retries, Idempotency, Logging

### Key Goals

✅ **חוויית צ'אט טבעית**: שאלות כמו "מה דחוף היום", "סכם שרשור", "תכין טיוטת תשובה"

✅ **בטיחות מקסימלית**: 
- אין שליחה אוטומטית ב-MVP
- אישור חובה לפעולות הרסניות
- Preview לפני ביצוע

✅ **איכות הנדסית**:
- אין פעולות כפולות (Idempotency)
- כשל לא מפיל מערכת
- כל פעולה נרשמת ב-Audit log
- מינימום מידע נשלח ל-AI

---

## 🏗️ System Architecture

### High-Level Architecture

```
┌─────────────┐
│   User      │
│  (Telegram) │
└──────┬──────┘
       │
       │ 1. Message
       ▼
┌─────────────────────────────────────┐
│           n8n Workflows              │
│  ┌──────────────────────────────┐  │
│  │  Chat Ingest & Decide        │  │
│  │  - Load Session              │  │
│  │  - Build AI Input            │  │
│  └──────────┬───────────────────┘  │
│             │                       │
│             │ 2. Minimal Context    │
│             ▼                       │
│  ┌──────────────────────────────┐  │
│  │     OpenAI Agent              │  │
│  │  - Intent Detection           │  │
│  │  - Parameter Extraction       │  │
│  │  - Returns JSON Decision      │  │
│  └──────────┬───────────────────┘  │
│             │                       │
│             │ 3. JSON Decision      │
│             ▼                       │
│  ┌──────────────────────────────┐  │
│  │  Validation & Safety Layer   │  │
│  │  - Schema Validation          │  │
│  │  - Confirmation Check         │  │
│  │  - Idempotency Check          │  │
│  └──────────┬───────────────────┘  │
│             │                       │
│             ├─ needs_confirmation? │
│             │   YES → Preview       │
│             │   NO → Execute        │
│             ▼                       │
│  ┌──────────────────────────────┐  │
│  │  Gmail Operations             │  │
│  │  - Search, Get, Thread        │  │
│  │  - Label, Archive, Mark Read  │  │
│  │  - Create Draft               │  │
│  └──────────┬───────────────────┘  │
│             │                       │
│             │ 4. Results            │
│             ▼                       │
│  ┌──────────────────────────────┐  │
│  │  Logging & Response           │  │
│  │  - Audit Log                  │  │
│  │  - Session Update             │  │
│  │  - User Response              │  │
│  └──────────────────────────────┘  │
└─────────────────────────────────────┘
       │
       │ 5. Formatted Response
       ▼
┌─────────────┐
│    User     │
│  (Telegram) │
└─────────────┘
```

### Component Breakdown

#### 1. Chat Interface
- **Telegram Bot** (MVP primary)
- **Webhook Endpoint** (alternative)
- Future: Web UI

#### 2. n8n Orchestration Layer
- Workflow orchestration
- Safety & validation
- Gmail API integration
- State management

#### 3. AI Agent (OpenAI)
- GPT-4 or GPT-3.5-turbo
- Structured JSON output
- Intent classification
- Content summarization

#### 4. Gmail API
- OAuth 2.0 authentication
- Scopes: `gmail.readonly`, `gmail.modify`, `gmail.compose`
- **NOT** `gmail.send` - no automatic sending

#### 5. Data Storage
- **n8n Data Store** (MVP)
- **PostgreSQL** (Production)
- Tables: Session, Pending Actions, Idempotency, Audit

---

## 🎨 Features & Capabilities

### Core Capabilities

#### 1. Email Search 🔍
- Gmail query syntax support
- Default limits (5-10 results)
- Sorted by newest
- Returns: id, subject, from, date, snippet

#### 2. Email Reading & Summarization 📧
- Latest unread summary
- Read specific email by ID
- Thread summarization
- HTML cleaning & quoted text removal
- Urgency scoring (1-5)
- Category classification
- Action items extraction

#### 3. Email Management 🏷️
- Apply/remove labels
- Archive emails
- Mark as read
- Batch operations with dry run

#### 4. Draft Creation ✍️
- Generate reply drafts
- Context-aware responses
- Preview before creation
- Editable suggestions
- Professional structure (greeting + body + CTA)

#### 5. Safety Layer 🔒
- Mandatory confirmation for destructive actions
- Dry run for bulk operations
- Idempotency (no duplicate actions)
- Retry with exponential backoff
- Comprehensive audit logging

#### 6. Privacy Features 🛡️
- Minimal data to AI (subject, from, date, snippet)
- Optional PII redaction (phone, ID, email)
- HTML stripping
- No body storage by default
- Least privilege OAuth scopes

---

## 🗺️ Complete Road Map

### Phase 1: Foundation & Infrastructure Setup
**Duration: 3-5 days | Risk: Medium | Priority: Critical**

#### Tasks:
- [ ] **1.1** Install n8n (Docker Compose recommended)
- [ ] **1.2** Setup Data Store / PostgreSQL
- [ ] **1.3** Configure environment variables
- [ ] **1.4** Create Telegram bot via BotFather
- [ ] **1.5** Setup OpenAI credentials in n8n
- [ ] **1.6** Setup Gmail OAuth 2.0 credentials
  - Create Google Cloud Project
  - Enable Gmail API
  - Configure OAuth consent screen
  - Generate Client ID & Secret
  - Set redirect URI
  - Configure scopes: `gmail.readonly`, `gmail.modify`, `gmail.compose`
- [ ] **1.7** Test end-to-end connection (ping-pong workflow)

**Deliverables:**
✅ Working n8n instance  
✅ Telegram bot responds to messages  
✅ All credentials configured  
✅ Basic workflow executable  

---

### Phase 2: Gmail Integration Layer
**Duration: 4-6 days | Risk: Medium | Priority: High**

#### Tasks:
- [ ] **2.1** Gmail Search Node
  - Parameters: query, limit, sort
  - Output: [{id, threadId, subject, from, date, snippet}]
  - Tests: unread, from:user, empty results
  
- [ ] **2.2** Gmail Get Email Node
  - Input: email_id
  - Output: full email object with body
  - Handle HTML-heavy emails
  
- [ ] **2.3** Gmail Get Thread Node
  - Input: thread_id
  - Output: sorted messages array
  - Identify last reply
  
- [ ] **2.4** Gmail Create Draft Node
  - Input: threadId, to, subject, body
  - **Never send** - only create draft
  - Return: draftId
  
- [ ] **2.5** Gmail Modify Labels/Archive/Mark Read
  - Single email operations
  - Batch operations support
  - Error handling per item
  
- [ ] **2.6** Error Handling & Retries
  - 429 (rate limit) → exponential backoff
  - 5xx (server error) → retry max 3 times
  - 401 (auth) → refresh token
  - Friendly error messages

**Deliverables:**
✅ All 6 Gmail operations functional  
✅ Graceful error handling  
✅ Detailed operation logs  

---

### Phase 3: AI Agent - Decision Brain
**Duration: 5-7 days | Risk: High | Priority: Critical**

#### Tasks:
- [ ] **3.1** Write comprehensive System Prompt
  - Role definition
  - Intent list
  - Guardrails rules
  - JSON schema
  
- [ ] **3.2** Define JSON Output Schema
```json
{
  "intent": "search|summarize|read|draft_reply|label|archive|mark_read|ask_clarifying",
  "confidence": 0.0-1.0,
  "needs_confirmation": boolean,
  "message_to_user": "string (Hebrew)",
  "email_insights": {
    "sentiment": "positive|neutral|negative|mixed",
    "urgency": 1-5,
    "category": "string",
    "summary": "string",
    "action_items": {
      "exists": boolean,
      "items": ["string"]
    }
  },
  "action": {
    "type": "gmail.*",
    "params": {...}
  }
}
```

- [ ] **3.3** Build JSON Validation Node
  - Schema validation
  - Required fields check
  - Range validation (confidence, urgency)
  - On failure: no execution + user message + log
  
- [ ] **3.4** Implement Guardrails
  - Confidence < 0.70 → ask_clarifying
  - Destructive actions → needs_confirmation: true
  - Bulk operations → needs_confirmation: true
  - Draft creation → needs_confirmation: true
  - No hallucination allowed
  - Never claim execution without confirmation
  
- [ ] **3.5** Intent Detection Testing
  - Create test matrix (10+ test cases)
  - Run tests and document results
  - Validate confidence scores
  - Validate confirmation requirements

**Deliverables:**
✅ 100% valid JSON responses  
✅ Invalid responses blocked  
✅ Guardrails operational  
✅ Test results documented  

---

### Phase 4: Core Workflows Implementation
**Duration: 8-12 days | Risk: High | Priority: Critical**

#### WF1: Chat Ingest & Decide
**Nodes:**
1. Telegram Trigger
2. DataStore: Load Session
3. Set: Build AI Input
4. OpenAI: Decide Intent
5. Code: Validate JSON Schema
6. Switch: Route by Intent
7. IF needs_confirmation → Preview + Save pending_action
8. ELSE → Execute
9. Error Handler

**Deliverable:** Complete decision pipeline

---

#### WF2: Gmail Search Execution
**Nodes:**
1. Gmail: Search Emails
2. Code: Clean & Redact Snippets
3. Code: Format Results
4. Send Response
5. DataStore: Save Results

**Deliverable:** Working search with clean output

---

#### WF3: Summarize Latest Unread
**Nodes:**
1. Call WF2 (is:unread, limit 10)
2. Loop: Get Each Email
3. Code: Clean HTML + Strip Quoted
4. OpenAI: Batch Summarize
5. Format Response (urgency + category + action)
6. Send to User
7. Log

**Deliverable:** Unread summary with insights

---

#### WF4: Get Email by ID
**Nodes:**
1. Gmail: Get Email
2. Code: Clean Body + Strip Quoted
3. Optional: OpenAI Summarize
4. Format with Insights
5. Send Response

**Deliverable:** Single email read with analysis

---

#### WF5: Summarize Thread
**Nodes:**
1. Gmail: Get Thread
2. Code: Strip Quoted from Each Message
3. OpenAI: Summarize Thread
4. Format Response
5. Send to User

**Deliverable:** One-line thread summary with action items

---

#### WF6: Draft Reply (Preview → Confirm → Create)
**Nodes:**
1. Gmail: Get Email/Thread (context)
2. OpenAI: Generate Draft
   - Structure: Greeting + Body + CTA + Placeholder Signature
3. Build Preview Message
4. Save pending_action to DataStore
5. Send Preview
6. **Wait for confirmation** (WF8)

**Deliverable:** Draft preview shown, not created until confirmation

---

#### WF7: Bulk Label & Archive (Dry Run → Confirm → Execute)
**Nodes:**
1. Gmail: Search Emails
2. Code: Build Dry Run Preview (count + samples)
3. Send Preview
4. Save pending_action
5. **Wait for confirmation** (WF8)

**Deliverable:** Dry run displayed before execution

---

#### WF8: Confirmation Handler
**Nodes:**
1. Telegram: Receive "אשר"
2. DataStore: Load pending_action
3. IF no pending → Reply "no action pending"
4. Code: Re-validate + Idempotency Check
5. Switch by action_type
6. Execute Action (loop if batch)
7. DataStore: Clear pending_action
8. Build Result Summary
9. Send to User
10. Log to Audit

**Deliverable:** Confirmation gate works, actions execute only after approval

---

**Phase 4 Success Criteria:**
✅ All 7 workflows operational end-to-end  
✅ Mandatory confirmation for destructive actions  
✅ Dry run for bulk operations  
✅ Preview for draft creation  

---

### Phase 5: Data Layer (Session, Idempotency, Audit)
**Duration: 3-5 days | Risk: Medium | Priority: High**

#### Tasks:

**5.1 Session State Table**
```sql
CREATE TABLE session_state (
  user_id VARCHAR(255) PRIMARY KEY,
  last_intent VARCHAR(100),
  last_email_ids TEXT[],
  last_context JSONB,
  preferences JSONB DEFAULT '{"default_limit": 10, "redaction_enabled": true, "language": "he"}',
  updated_at TIMESTAMP DEFAULT NOW()
);
```
- [ ] Create in DataStore/Postgres
- [ ] TTL: 24 hours
- [ ] Nodes: Load Session, Save Session

---

**5.2 Pending Actions Table**
```sql
CREATE TABLE pending_actions (
  user_id VARCHAR(255) PRIMARY KEY,
  action_type VARCHAR(100),
  params JSONB,
  created_at TIMESTAMP DEFAULT NOW(),
  expires_at TIMESTAMP
);
```
- [ ] Create in DataStore
- [ ] TTL: 15 minutes
- [ ] Nodes: Save Pending, Load Pending, Clear Pending

---

**5.3 Idempotency Keys Table**
```sql
CREATE TABLE idempotency_keys (
  key VARCHAR(255) PRIMARY KEY,
  user_id VARCHAR(255),
  action_type VARCHAR(100),
  executed_at TIMESTAMP DEFAULT NOW(),
  result JSONB
);
```
- [ ] Create in DataStore
- [ ] Key generation: `hash(user_id + action_type + email_id + params)`
- [ ] Check before execution → if exists, return cached result
- [ ] TTL: 7 days

---

**5.4 Audit Log Table**
```sql
CREATE TABLE audit_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  timestamp TIMESTAMP DEFAULT NOW(),
  user_id VARCHAR(255),
  intent VARCHAR(100),
  action_type VARCHAR(100),
  params_summary JSONB,
  execution_confirmed BOOLEAN,
  success BOOLEAN,
  error_code VARCHAR(50),
  error_message TEXT
);
```
- [ ] Create in DataStore/Postgres
- [ ] Log every action attempt
- [ ] **No full email bodies** - only minimal params
- [ ] Node: Log Action (called from all execution workflows)

---

**5.5 Testing**
- [ ] Test idempotency: duplicate action → single execution
- [ ] Test pending expiration: 15 min timeout
- [ ] Test audit trail: all actions logged and queryable

**Deliverables:**
✅ No duplicate actions  
✅ Complete audit trail  
✅ Session state persists between messages  

---

### Phase 6: Security & Privacy Layer
**Duration: 3-4 days | Risk: High | Priority: Critical**

#### Tasks:

**6.1 Content Cleaning & Redaction**

- [ ] **Code Node: `cleanHTML(text)`**
```javascript
function cleanHTML(html) {
  // Remove all HTML tags
  let text = html.replace(/<[^>]*>/g, '');
  // Decode HTML entities
  text = text.replace(/&nbsp;/g, ' ').replace(/&amp;/g, '&');
  // Truncate to 500 chars for snippets
  if (text.length > 500) text = text.substring(0, 500) + '...';
  return text.trim();
}
```

- [ ] **Code Node: `stripQuotedHistory(body)`**
```javascript
function stripQuotedHistory(body) {
  // Remove lines starting with >
  let lines = body.split('\n');
  lines = lines.filter(line => !line.trim().startsWith('>'));
  
  // Remove "On [date], [name] wrote:" blocks
  body = lines.join('\n');
  body = body.replace(/On .+? wrote:/gi, '');
  
  return body.trim();
}
```

- [ ] **Code Node: `redactPII(text)` (if enabled)**
```javascript
function redactPII(text) {
  // Phone numbers: XXX-XXXXXXX or XXXXXXXXXX
  text = text.replace(/\d{3}-?\d{7}/g, '[PHONE]');
  
  // Israeli ID: 9 digits
  text = text.replace(/\b\d{9}\b/g, '[ID]');
  
  // Optional: email addresses
  // text = text.replace(/[\w.-]+@[\w.-]+\.\w+/g, '[EMAIL]');
  
  return text;
}
```

---

**6.2 Minimal Data to AI**

**Policy:** Only send to OpenAI:
- `subject`
- `from`
- `date`
- `snippet` (max 500 chars, cleaned)

If full body needed: clean + redact first

- [ ] Create filter node before all OpenAI calls
- [ ] Verify in logs: no full bodies sent to AI

---

**6.3 Confirmation Gate Enforcement**

- [ ] **Code Node: `requiresConfirmation(action)`**
```javascript
function requiresConfirmation(action, params) {
  const destructive = ['archive', 'mark_read', 'label', 'draft_reply'];
  const isBulk = params.query || (params.email_ids && params.email_ids.length > 1);
  
  return destructive.includes(action.type) || isBulk;
}
```

- [ ] IF requires confirmation AND no confirmation given:
  - **BLOCK** execution
  - Send preview/dry run
  - Request confirmation
  - Log attempt

---

**6.4 Least Privilege OAuth Scopes**

**Required Scopes:**
✅ `https://www.googleapis.com/auth/gmail.readonly` - search, get  
✅ `https://www.googleapis.com/auth/gmail.modify` - labels, archive, mark read  
✅ `https://www.googleapis.com/auth/gmail.compose` - draft only  

**Forbidden:**
❌ `https://www.googleapis.com/auth/gmail.send` - NO automatic sending

- [ ] Review OAuth configuration
- [ ] Test: attempt to send email → should fail

---

**6.5 No Body Storage by Default**

- [ ] Session State: do NOT store `body` fields
- [ ] Audit Log: only store minimal `params_summary`:
```json
{
  "email_id": "abc123",
  "action": "archive",
  "count": 5
}
```
- [ ] User Preferences: toggle `save_bodies` (default: false)

---

**Phase 6 Success Criteria:**
✅ No full bodies in logs (unless user enabled)  
✅ PII removed/masked before AI processing  
✅ No destructive actions without confirmation  
✅ Minimal OAuth scopes only  

---

### Phase 7: Testing & Validation
**Duration: 5-7 days | Risk: Medium | Priority: High**

#### Test Plan (Minimum 10 Scenarios)

---

#### Test 1: Summarize Latest Unread (Happy Path)
**Input:** "מה דחוף היום"

**Expected:**
- AI decision: `summarize`, confidence > 0.70, needs_confirmation: false
- Gmail search: `is:unread`, limit 10
- Response: list of summaries with urgency 1-5

**Evidence:** Screenshot + execution log

---

#### Test 2: Search Emails with Query
**Input:** "חפש מיילים מג'ון על חשבונית"

**Expected:**
- AI decision: `search`, query: `from:john invoice`
- Gmail returns results
- Response: formatted result list

**Evidence:** Execution log

---

#### Test 3: Read Specific Email
**Input:** "קרא מייל [email_id]"

**Expected:**
- AI decision: `read`, email_id provided
- Gmail get email
- Response: summary + insights (urgency, category, action items)

**Evidence:** Log

---

#### Test 4: Summarize Thread
**Input:** "סכם את השרשור"

**Expected:**
- AI decision: `summarize`, thread_id
- Gmail get thread
- OpenAI summarize
- Response: 2-3 line summary + action items

**Evidence:** Log

---

#### Test 5: Draft Reply with Confirmation
**Input:** "הכן תשובה לאימייל האחרון - אשר וציין 2 זמנים"

**Expected:**
1. AI decision: `draft_reply`, needs_confirmation: true
2. Preview sent to user
3. User sends "אשר"
4. Draft created in Gmail (verified in Gmail UI)

**Evidence:** Log + Gmail screenshot showing draft

---

#### Test 6: Bulk Label & Archive with Dry Run
**Input:** "תייג הכל כניוזלטר וארכב"

**Expected:**
1. AI decision: `label` + `archive`, needs_confirmation: true
2. Dry run: "נמצאו 47 ניוזלטרים" + 3 examples
3. User confirms
4. Actions applied (verified in Gmail)

**Evidence:** Log + before/after Gmail screenshots

---

#### Test 7: Invalid JSON from AI
**Setup:** Craft prompt that causes AI to return malformed JSON

**Expected:**
- Validation fails
- **No actions executed**
- User message: "לא הצלחתי להבין, נסח מחדש"
- Error logged with details

**Evidence:** Log showing validation failure + no Gmail API calls

---

#### Test 8: Rate Limit (429) Handling
**Setup:** Trigger rate limit (simulate or force with rapid requests)

**Expected:**
- Retry with exponential backoff (1s → 2s → 4s)
- Max 3 retries
- If still fails: user message + error log

**Evidence:** Log showing retry attempts with timestamps

---

#### Test 9: HTML Heavy Email
**Setup:** Email with complex HTML + inline CSS + images

**Expected:**
- HTML cleaning works
- Plain text extracted
- Snippet truncated to 500 chars
- No HTML tags in response

**Evidence:** Log showing before/after cleaning

---

#### Test 10: Destructive Action Without Confirmation Blocked
**Setup:** Attempt to bypass confirmation gate (simulate by setting needs_confirmation: false manually)

**Expected:**
- Action **blocked** by n8n validation
- No Gmail API call made
- User message: "פעולה זו דורשת אישור"
- Attempt logged in audit

**Evidence:** Log showing block + no Gmail execution

---

#### Test Results Documentation

- [ ] Create results table:

| Test # | Description | Result | Log/Screenshot Link |
|--------|-------------|--------|---------------------|
| 1 | Summarize unread | ✅ PASS | [log-001.json] |
| 2 | Search query | ✅ PASS | [log-002.json] |
| 3 | Read email | ✅ PASS | [log-003.json] |
| 4 | Summarize thread | ✅ PASS | [log-004.json] |
| 5 | Draft with confirm | ✅ PASS | [log-005.json, screenshot-001.png] |
| 6 | Bulk with dry run | ✅ PASS | [log-006.json, screenshot-002.png] |
| 7 | Invalid JSON | ✅ PASS | [log-007.json] |
| 8 | Rate limit retry | ✅ PASS | [log-008.json] |
| 9 | HTML cleaning | ✅ PASS | [log-009.json] |
| 10 | Block no-confirm | ✅ PASS | [log-010.json] |

- [ ] Save execution logs for each scenario
- [ ] Take screenshots of Telegram conversations
- [ ] Document any edge cases discovered

**Phase 7 Success Criteria:**
✅ 10/10 test scenarios pass  
✅ All edge cases documented  
✅ Complete logs available  

---

### Phase 8: Documentation & Release
**Duration: 2-3 days | Risk: Low | Priority: Medium**

#### Tasks:

**8.1 Export Workflows**
- [ ] Export all workflows as JSON from n8n
- [ ] Organize files:
```
/workflows
  ├── WF1_chat_ingest_decide.json
  ├── WF2_gmail_search.json
  ├── WF3_summarize_unread.json
  ├── WF4_get_email.json
  ├── WF5_summarize_thread.json
  ├── WF6_draft_reply.json
  ├── WF7_bulk_label_archive.json
  └── WF8_confirmation_handler.json
```

---

**8.2 README.md**

**Content:**
- [ ] System overview
- [ ] Prerequisites (n8n, Postgres/DataStore, Telegram, OpenAI, Gmail)
- [ ] Installation instructions:
  - Docker Compose setup
  - Environment variables
  - Credentials configuration
  - Workflow import
- [ ] Basic usage:
  - Connect Gmail account
  - Send messages to bot
  - Confirm actions
- [ ] Architecture diagram
- [ ] Limitations:
  - No sending in MVP
  - AI can make mistakes
  - Gmail rate limits
- [ ] Troubleshooting:
  - Common errors
  - How to check logs
  - How to reset session

---

**8.3 ARCHITECTURE.md**
- [ ] Detailed component diagrams (text-based or Mermaid)
- [ ] Explanation of each workflow
- [ ] Data layer details
- [ ] Security & privacy implementation

---

**8.4 TESTING.md**
- [ ] Complete test case list
- [ ] Test results
- [ ] Links to logs and evidence

---

**8.5 LICENSE & CONTRIBUTION**
- [ ] LICENSE file (MIT/Apache/proprietary)
- [ ] CONTRIBUTING.md (if open source)

---

**Phase 8 Success Criteria:**
✅ All deliverables complete  
✅ New user can setup from README  
✅ Documentation clear and comprehensive  

---

## 📊 Timeline & Resource Estimation

| Phase | Duration (days) | Risk | Dependencies |
|-------|----------------|------|--------------|
| 1. Foundation | 3-5 | Medium | - |
| 2. Gmail Integration | 4-6 | Medium | Phase 1 |
| 3. AI Agent | 5-7 | **High** | Phase 1 |
| 4. Core Workflows | 8-12 | **High** | Phase 2, 3 |
| 5. Data Layer | 3-5 | Medium | Phase 4 |
| 6. Security & Privacy | 3-4 | **High** | Phase 4, 5 |
| 7. Testing | 5-7 | Medium | Phase 6 |
| 8. Documentation | 2-3 | Low | Phase 7 |

**Total Estimated Time: 33-49 working days (6-9 weeks full-time)**

---

## 📝 Detailed Use Cases

### UC1: Summarize Latest Unread Emails

**Trigger:** User asks "מה דחוף היום" or "Summarize my latest unread emails"

**Input:**
- `chat_text`: user message
- `user_id`: Telegram user ID
- `limit` (optional, default 10)

**Flow:**
1. User sends message
2. n8n loads session preferences
3. AI detects intent: `summarize`, confidence: 0.82
4. AI generates action: `gmail.search` with `query: "is:unread"`, `limit: 10`
5. n8n validates JSON schema
6. n8n executes Gmail search
7. For each result: get email, clean HTML, strip quoted text
8. AI summarizes batch with urgency + category
9. n8n formats response with emoji indicators
10. Send to user

**Output:**
```
📧 **5 מיילים לא נקראים:**

1. [🔴 דחוף 5/5] מג'ון סמית' - חשבונית דורשת תשלום
   📂 כספים | 😰 לחץ
   📌 פעולה: שלח אישור עד מחר 5 אחה"צ

2. [🟡 רגיל 3/5] מסרה כהן - עדכון פרויקט שבועי
   📂 עבודה | 😊 חיובי
   📌 פעולה: קרא לפני פגישת יום ד'

3. [🟢 נמוך 2/5] מניוזלטר - המלצות שבועיות
   📂 ניוזלטר | 😐 ניטרלי
   📌 פעולה: קרא בשעות פנאי
```

**Success Criteria:**
- Works even with HTML-heavy emails
- No full body exposed
- Respects limit parameter
- Urgency scoring makes sense

**Edge Cases:**
- No unread emails → "אין מיילים לא נקראים"
- Very short email → still generate valid summary
- HTML body only → clean and extract text

---

### UC2: Search Emails by Query

**Trigger:** "חפש מיילים מג'ון על חשבונית" or "Find emails from X about Y"

**Input:**
- `chat_text`: search request
- `query` (extracted by AI): Gmail syntax

**Flow:**
1. AI detects intent: `search`
2. AI extracts query: `from:john subject:חשבונית OR body:חשבונית`
3. AI sets limit: 10 (default)
4. n8n validates
5. Gmail search executed
6. Results cleaned & formatted
7. AI recommends next step

**Output:**
```
🔍 **נמצאו 3 תוצאות:**

1. מ: john@example.com
   נושא: חשבונית מס' 12345 לתשלום
   תאריך: 28 ינואר 2026
   📄 "הי, מצורפת חשבונית לפרויקט..."

2. מ: john.smith@company.com
   נושא: Re: חשבונית רבעון ראשון
   תאריך: 15 ינואר 2026
   📄 "תודה על האישור, הנה הפירוט..."

3. מ: john@startup.io
   נושא: חשבונית עבור ייעוץ דצמבר
   תאריך: 5 ינואר 2026
   📄 "שלום, כמוסכם הנה החשבונית..."

💡 **המלצה:** רוב התוצאות קשורות לתשלומים. רוצה שאסכם את ההודעה הראשונה?
```

**Success Criteria:**
- Query not empty
- Sorted by newest
- Limit enforced
- Helpful next step suggestion

**Edge Cases:**
- Ambiguous query, low confidence → ask clarification
- Too many results → show first N, offer pagination (future)
- Rate limit → retry with backoff

---

### UC3: Read Specific Email by ID

**Trigger:** User selects from search results or asks "read email id abc123"

**Input:**
- `email_id`

**Flow:**
1. AI detects intent: `read`
2. n8n validates email_id
3. Gmail Get Email
4. Clean HTML + strip quoted text
5. AI analyzes:
   - Sentiment
   - Urgency (1-5)
   - Category
   - Action items
   - Summary
6. Format and send

**Output:**
```
📧 **מייל מ: john@example.com**

**נושא:** חשבונית מס' 12345 לתשלום  
**תאריך:** 28 ינואר 2026, 14:35  

**תקציר:**
ג'ון מבקש לשלם חשבונית עבור פרויקט שהסתיים בדצמבר. הסכום: ₪5,000. 
דדליין לתשלום: 5 פברואר. הוא מצרף קובץ PDF עם הפירוט.

📊 **ניתוח:**
- 🎯 דחיפות: 5/5 (דדליין קרוב)
- 😰 סנטימנט: לחץ (תשלום דחוף)
- 📂 קטגוריה: כספים / חשבונות

✅ **פעולות נדרשות:**
1. אשר את החשבונית
2. העבר לחשבונאית עד 31 ינואר
3. שלח אישור לג'ון

💬 **רוצה שאכין טיוטת תשובה?**
```

**Success Criteria:**
- No raw HTML displayed
- Redaction applied if enabled
- Insights are accurate

**Edge Cases:**
- email_id doesn't exist → "לא נמצא מייל"
- No body (attachment only) → "המייל מכיל רק קבצים מצורפים"

---

### UC4: Summarize Thread

**Trigger:** "סכם את השרשור" or "Summarize this thread"

**Input:**
- `thread_id` or derive from email_id

**Flow:**
1. AI detects intent: `summarize` with thread context
2. Gmail Get Thread
3. For each message: strip quoted history
4. AI summarizes entire conversation
5. Extract action items
6. Determine urgency and next action

**Output:**
```
🧵 **סיכום שרשור: "פגישה לגבי הצעת מחיר"**

**משתתפים:** אתה, sarah@client.com, dan@team.com  
**הודעות:** 7 | **תאריך אחרון:** היום 11:30

**תקציר:**
סרה ביקשה הצעת מחיר לפרויקט חדש. דן שלח טיוטה ראשונית. סרה ביקשה הבהרות 
לגבי המחיר וטיימליין. דן ענה שהמחיר סופי אבל ניתן לגמישות בזמנים. סרה מחכה 
להחלטה סופית שלך.

📌 **פעולות נדרשות:**
- [ ] החלט האם לאשר את ההצעה
- [ ] אם כן, שלח אישור לסרה
- [ ] תאם עם דן את תאריך ההתחלה

🎯 **דחיפות:** 4/5 (הלקוח מחכה להחלטה)  
👉 **צעד הבא:** להחליט עד סוף השבוע

💬 **רוצה שאכין טיוטת תשובה לסרה?**
```

**Success Criteria:**
- Quoted history removed
- One coherent summary
- Clear action items
- Next step identified

**Edge Cases:**
- Very long thread (20+ messages) → summarize last 10 only (MVP)
- Mixed languages → handle gracefully
- Newsletter in middle of thread → filter out

---

### UC5: Create Draft Reply with Preview & Confirmation

**Trigger:** "הכן תשובה לאימייל האחרון" or "Draft a reply saying yes and propose 2 times"

**Input:**
- Context from last email/thread
- User instructions

**Flow:**
1. AI detects intent: `draft_reply`
2. AI sets needs_confirmation: true (always for drafts)
3. Gmail Get Email/Thread for context
4. AI generates draft:
   - Greeting
   - Clear response
   - Question/CTA for next step
   - Placeholder signature
5. n8n builds preview
6. Send preview to user
7. Save pending_action
8. **Wait for "אשר"**
9. On confirmation: Gmail Create Draft
10. Send success message with link to Gmail

**Preview Output:**
```
✍️ **טיוטת תשובה מוכנה:**

━━━━━━━━━━━━━━━━━━━━
Hi Sarah,

Thank you for your patience. I've reviewed Dan's proposal and I'm happy to move forward with the project.

Regarding the timeline, would either of these dates work for you?
- Option 1: Starting February 15th
- Option 2: Starting March 1st

Please let me know your preference, and we'll finalize the agreement.

Best regards,
[Your Name]
━━━━━━━━━━━━━━━━━━━━

📝 **פעולות:**
✅ שלח "אשר" ליצירת הטיוטה ב-Gmail
✏️ או כתב הערות לעריכה
❌ או "בטל"
```

**After Confirmation:**
```
✅ **הטיוטה נוצרה בהצלחה!**

📬 הטיוטה נשמרה ב-Gmail שלך ומצורפת לשרשור עם Sarah.

🔗 [פתח ב-Gmail](https://mail.google.com/mail/u/0/#drafts?compose=abc123)

💡 **טיפ:** עבור ל-Gmail כדי לערוך ולשלוח.
```

**Success Criteria:**
- Never sends automatically
- Always shows preview
- Requires explicit confirmation
- Professional structure

**Edge Cases:**
- User doesn't provide critical details (e.g., 2 times) → ask clarifying question
- Inappropriate tone → offer alternative or ask for guidance
- AI returns invalid JSON → block creation, ask rephrase

---

### UC6: Bulk Label & Archive Newsletters with Confirmation

**Trigger:** "תייג הכל כניוזלטר וארכב" or "Label all newsletters as Newsletter and archive them"

**Input:**
- Query to identify newsletters (AI-generated or user-provided)
- Label target
- Limit/batch size

**Flow:**
1. AI detects intent: `label` + `archive`
2. AI generates query: `from:(newsletter OR unsubscribe OR marketing)`
3. AI sets needs_confirmation: true (bulk operation)
4. Gmail Search with query, limit 100
5. n8n builds dry run preview:
   - Total count
   - 3-5 examples
6. Send preview
7. Save pending_action
8. **Wait for "אשר"**
9. On confirmation:
   - Loop through results
   - For each: apply label + archive
   - Check idempotency per email_id
10. Build result summary
11. Send to user
12. Log all actions

**Dry Run Output:**
```
📊 **תצוגה מקדימה - Dry Run**

🔍 נמצאו **47 ניוזלטרים** התואמים לחיפוש

📌 **דוגמאות:**
1. Marketing Weekly מאת newsletter@company.com
   "Your weekly dose of marketing tips..."
   
2. Tech Digest מאת updates@techblog.io
   "Top 10 tech news this week..."
   
3. Daily Sales Report מאת reports@sales-platform.com
   "Here's your daily sales summary..."

⚡ **פעולה מתוכננת:**
- תייג כל 47 מיילים עם התווית "Newsletter"
- ארכב את כולם (יוסרו מתיבת הדואר הנכנס)

⚠️ **אזהרה:** פעולה זו תשנה 47 מיילים.

✅ שלח "אשר" להמשך
❌ או "בטל" לביטול
```

**After Confirmation:**
```
✅ **הפעולה הושלמה!**

📊 **סיכום:**
- ✅ 47 מיילים תויגו כ-"Newsletter"
- ✅ 47 מיילים אורכבו
- ❌ 0 כשלונות

⏱️ **זמן ביצוע:** 8 שניות

💡 **טיפ:** תוכל למצוא אותם בתווית "Newsletter" בעתיד.
```

**Success Criteria:**
- Confirmation mandatory
- Dry run shows count + samples
- Idempotency prevents duplicates
- Each action logged

**Edge Cases:**
- Newsletter detection uncertain → suggest query or rule
- Label doesn't exist → create or ask user
- Duplicate execution attempt → idempotency key blocks

---

### UC7: Mark Read / Archive / Label Operations

**Trigger:** "סמן כנקרא", "ארכב", "תייג כחשוב"

**Input:**
- `email_id` or list of IDs or query
- Action type

**Flow:**
1. AI detects intent: `mark_read` / `archive` / `label`
2. IF bulk (query or multiple IDs):
   - needs_confirmation: true
   - Dry run
3. ELSE single email:
   - needs_confirmation: true (destructive action)
   - Quick preview
4. n8n validates params
5. **Wait for confirmation**
6. Execute action(s)
7. Log
8. Send result

**Output:**
```
⚠️ **אישור נדרש**

פעולה: סימון כנקרא  
מספר מיילים: 1

📧 מייל: "חשבונית מס' 12345 לתשלום" מאת john@example.com

✅ שלח "אשר" לביצוע
❌ או "בטל"
```

**Success Criteria:**
- Block without confirmation
- Validate parameters
- Single email: quick confirm
- Bulk: dry run first

**Edge Cases:**
- Attempt without confirmation → blocked + message
- Missing limit in bulk → default to 10 or ask
- Invalid email_id → error message

---

### UC8: Recovery from Invalid AI JSON

**Trigger:** AI returns malformed JSON

**Input:**
- `raw_response` from OpenAI (invalid JSON)

**Flow:**
1. AI node returns response
2. Code validation node parses JSON
3. Parsing fails or schema validation fails
4. Catch error
5. Log:
   - Timestamp
   - User ID
   - Chat text
   - Raw AI response
   - Error details
6. Send message to user: "לא הצלחתי להבין את בקשתך. אנא נסח מחדש או נסה שוב."
7. **No Gmail actions executed**
8. Continue workflow (don't crash)

**Output:**
```
😕 **לא הצלחתי להבין**

נראה שהיתה בעיה בהבנת הבקשה שלך. 

💬 אנא נסה:
- לנסח את הבקשה אחרת
- להיות יותר ספציפי
- לבקש פעולה אחת בכל פעם

דוגמאות לפקודות שאני מבין:
• "מה דחוף היום"
• "חפש מיילים מג'ון"
• "סכם את השרשור"
• "הכן תשובה"
```

**Success Criteria:**
- Zero Gmail operations on error
- System continues to work
- Helpful error message
- Detailed log for debugging

**Edge Cases:**
- Happens mid-flow after fetch → still block execution
- Repeated failures → might indicate prompt issue

---

### UC9: Rate Limit & Retry Handling

**Trigger:** Gmail API returns 429 or 5xx

**Input:**
- Gmail API error response

**Flow:**
1. Gmail node makes request
2. Response: 429 (rate limit) or 500/503 (server error)
3. Catch error
4. IF 429:
   - Wait 1 second
   - Retry (attempt 2)
   - If fails: wait 2 seconds
   - Retry (attempt 3)
   - If fails: wait 4 seconds
   - Retry (attempt 4 - final)
5. IF still fails OR max retries:
   - Log error with all attempts
   - Send user-friendly message
6. Continue workflow (don't crash entire system)

**Output to User:**
```
⏸️ **זמנית לא ניתן לבצע פעולה**

Gmail API חזר עם שגיאה זמנית. נסיתי 3 פעמים אבל עדיין לא הצלחתי.

🔄 **אפשרויות:**
- נסה שוב בעוד דקה
- הפעולה נשמרה ואפשר לנסות שוב מאוחר יותר

🆔 **מזהה שגיאה:** err_rate_limit_abc123
```

**Success Criteria:**
- Exponential backoff works
- No duplicate actions (idempotency)
- System doesn't crash
- User informed clearly

**Edge Cases:**
- Retry causes duplicate → idempotency key prevents
- Different error during retry → handle separately

---

## 🤖 AI Agent Specification

### System Prompt

```
You are an AI Email Assistant that helps users manage their Gmail inbox through conversation.

ROLE:
You analyze user requests and return ONLY structured JSON decisions. You NEVER execute actions yourself.

YOUR RESPONSIBILITIES:
1. Detect user intent from natural language (Hebrew or English)
2. Extract parameters needed for Gmail operations
3. Classify email content (urgency, category, sentiment)
4. Generate summaries and draft replies
5. Decide if user confirmation is required
6. NEVER hallucinate email content
7. NEVER claim you executed an action

AVAILABLE INTENTS:
- search: Search for emails matching a query
- summarize: Summarize unread emails or a thread
- read: Read a specific email by ID
- draft_reply: Generate a draft reply
- label: Apply or remove labels
- archive: Archive emails
- mark_read: Mark emails as read
- ask_clarifying: Request more information from user

GUARDRAILS (CRITICAL):
1. If confidence < 0.70 → return intent: "ask_clarifying"
2. ALWAYS set needs_confirmation: true for:
   - archive
   - mark_read
   - label (bulk)
   - draft_reply
   - Any operation affecting multiple emails
3. NEVER invent email content you didn't receive
4. NEVER claim "I have archived" or "I have sent" - you only recommend actions
5. If user request is unclear or ambiguous → ask_clarifying

RESPONSE FORMAT:
Return ONLY valid JSON matching this schema:

{
  "intent": "string (one of: search|summarize|read|draft_reply|label|archive|mark_read|ask_clarifying)",
  "confidence": number (0.0 to 1.0),
  "needs_confirmation": boolean,
  "message_to_user": "string in Hebrew - what you're about to do or question to ask",
  "email_insights": {
    "sentiment": "positive|neutral|negative|mixed",
    "urgency": number (1-5, where 5 is most urgent),
    "category": "string (e.g., עבודה, כספים, אישי, ניוזלטר)",
    "summary": "string (brief summary)",
    "action_items": {
      "exists": boolean,
      "items": ["string array of action items"]
    }
  },
  "action": {
    "type": "gmail.search|gmail.get|gmail.thread|gmail.draft|gmail.label|gmail.archive|gmail.mark_read",
    "params": {
      // Parameters specific to action type
      "query": "string (Gmail search syntax)",
      "email_id": "string",
      "thread_id": "string",
      "limit": number (default 10),
      "sort": "newest|oldest",
      "labelIds": ["string array"],
      "to": "string",
      "subject": "string",
      "body": "string"
    }
  }
}

EXAMPLES:

User: "מה דחוף היום"
Response:
{
  "intent": "summarize",
  "confidence": 0.85,
  "needs_confirmation": false,
  "message_to_user": "אסכם את המיילים הלא נקראו האחרונים שלך",
  "email_insights": {
    "sentiment": "neutral",
    "urgency": 3,
    "category": "כללי",
    "summary": "יופק אחרי שליפת המיילים",
    "action_items": {"exists": false, "items": []}
  },
  "action": {
    "type": "gmail.search",
    "params": {
      "query": "is:unread",
      "limit": 10,
      "sort": "newest"
    }
  }
}

User: "הכן תשובה לאימייל האחרון"
Response:
{
  "intent": "draft_reply",
  "confidence": 0.78,
  "needs_confirmation": true,
  "message_to_user": "אכין טיוטת תשובה למייל האחרון. תוכל לערוך אותה לפני השליחה.",
  "email_insights": {
    "sentiment": "neutral",
    "urgency": 3,
    "category": "עבודה",
    "summary": "הכנת טיוטה",
    "action_items": {"exists": false, "items": []}
  },
  "action": {
    "type": "gmail.draft",
    "params": {
      "thread_id": "[will be extracted from context]"
    }
  }
}

User: "תייג הכל כניוזלטר"
Response:
{
  "intent": "label",
  "confidence": 0.72,
  "needs_confirmation": true,
  "message_to_user": "אתייג את כל הניוזלטרים עם התווית 'Newsletter'. תקבל תצוגה מקדימה לפני הביצוע.",
  "email_insights": {
    "sentiment": "neutral",
    "urgency": 2,
    "category": "ניוזלטר",
    "summary": "תיוג ניוזלטרים",
    "action_items": {"exists": false, "items": []}
  },
  "action": {
    "type": "gmail.label",
    "params": {
      "query": "from:(newsletter OR unsubscribe) OR subject:(newsletter OR digest)",
      "labelIds": ["Newsletter"],
      "limit": 100
    }
  }
}

REMEMBER:
- Always respond in valid JSON
- Be conservative with confidence scores
- Prioritize user safety with confirmation gates
- Never execute - only recommend
- Handle Hebrew and English input
```

---

### JSON Schema (for Validation)

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["intent", "confidence", "needs_confirmation", "message_to_user", "action"],
  "properties": {
    "intent": {
      "type": "string",
      "enum": ["search", "summarize", "read", "draft_reply", "label", "archive", "mark_read", "ask_clarifying"]
    },
    "confidence": {
      "type": "number",
      "minimum": 0,
      "maximum": 1
    },
    "needs_confirmation": {
      "type": "boolean"
    },
    "message_to_user": {
      "type": "string",
      "minLength": 1
    },
    "email_insights": {
      "type": "object",
      "properties": {
        "sentiment": {
          "type": "string",
          "enum": ["positive", "neutral", "negative", "mixed"]
        },
        "urgency": {
          "type": "integer",
          "minimum": 1,
          "maximum": 5
        },
        "category": {
          "type": "string"
        },
        "summary": {
          "type": "string"
        },
        "action_items": {
          "type": "object",
          "properties": {
            "exists": {"type": "boolean"},
            "items": {
              "type": "array",
              "items": {"type": "string"}
            }
          },
          "required": ["exists", "items"]
        }
      }
    },
    "action": {
      "type": "object",
      "required": ["type", "params"],
      "properties": {
        "type": {
          "type": "string",
          "enum": ["gmail.search", "gmail.get", "gmail.thread", "gmail.draft", "gmail.label", "gmail.archive", "gmail.mark_read"]
        },
        "params": {
          "type": "object"
        }
      }
    }
  }
}
```

---

## 🔄 n8n Workflows Detail

### WF1: Chat Ingest & Decide

**Purpose:** Main entry point for all user messages

**Trigger:** Telegram Trigger or Webhook

**Nodes:**

1. **Telegram Trigger / Webhook**
   - Input: user message
   - Output: `message.text`, `message.from.id`

2. **DataStore: Load Session**
   - Input: `user_id`
   - Query: `SELECT * FROM session_state WHERE user_id = ?`
   - Output: `session` object (preferences, last_context, etc.)

3. **Set: Build AI Input**
   - Combine: `chat_text` + minimal `session` data
   - Output: prompt for AI

4. **OpenAI: Decide Intent**
   - Model: GPT-4 or GPT-3.5-turbo
   - System Prompt: (see AI Agent Specification)
   - User Message: from previous node
   - Response Format: JSON mode
   - Output: decision JSON

5. **Code: Validate JSON Schema**
   - Parse JSON
   - Validate against schema
   - IF invalid:
     - Log error
     - Send "לא הצלחתי להבין" to user
     - STOP workflow
   - IF valid: continue

6. **Switch: Check Confidence**
   - IF confidence < 0.70:
     - intent = "ask_clarifying"
   - Route to next node

7. **Switch: Route by Intent**
   - ask_clarifying → Send question to user → Save session → END
   - Other intents → Continue

8. **Code: Validate Action Params**
   - Check required params for action.type
   - IF missing critical params → ask_clarifying

9. **Switch: Check Confirmation Requirement**
   - IF needs_confirmation == true:
     - Route to Preview Builder
   - ELSE:
     - Route to Execution

10. **IF Preview Path:**
    - Code: Build Preview Message
    - Telegram: Send Preview
    - DataStore: Save pending_action
    - END (wait for user confirmation)

11. **IF Execute Path:**
    - Route to appropriate execution workflow (WF2-WF7)

12. **Error Handler:**
    - Catch all errors
    - Log with details
    - Send friendly error message
    - Don't crash

**Output:** Decision routed to execution or preview shown to user

---

### WF2: Gmail Search Execution

**Purpose:** Search emails and return formatted results

**Trigger:** Internal call from WF1

**Input:**
```json
{
  "query": "is:unread",
  "limit": 10,
  "sort": "newest"
}
```

**Nodes:**

1. **Gmail: Search Emails**
   - Use query param
   - Max results: limit
   - Order by: date descending if sort=newest

2. **Code: Clean & Redact Snippets**
   - For each result:
     - Strip HTML tags from snippet
     - Truncate to 500 chars
     - Apply redaction if enabled (phone, ID)

3. **Code: Format Results**
   - Build array of clean results
   - Each: {id, threadId, subject, from, date, snippet}

4. **Set: Prepare Response**
   - Format as user-friendly message
   - Add count, examples

5. **Telegram: Send Response**

6. **DataStore: Save to Session**
   - Update last_email_ids
   - Update last_context

7. **DataStore: Log Action**
   - Insert into audit_log

**Output:** Formatted search results sent to user

---

### WF3: Summarize Unread

**Purpose:** Get latest unread emails and provide summaries with urgency

**Trigger:** Internal call from WF1 (intent: summarize, is:unread)

**Nodes:**

1. **Execute Workflow: WF2**
   - query: "is:unread"
   - limit: 10

2. **Loop: For Each Email ID**
   - Get email IDs from WF2 results

3. **Gmail: Get Email**
   - Fetch full email by ID

4. **Code: Clean Body**
   - Strip HTML
   - Remove quoted history
   - Apply redaction

5. **Code: Build Batch for AI**
   - Collect all cleaned bodies
   - Create array: [{subject, from, body_snippet}, ...]

6. **OpenAI: Batch Summarize**
   - System: "Summarize each email in one line. Provide urgency 1-5, category, and suggested action."
   - Input: batch array
   - Output: array of summaries

7. **Code: Format Response**
   - For each summary:
     - Add urgency emoji (🔴 5, 🟡 3-4, 🟢 1-2)
     - Add category
     - Add action item
   - Build final message

8. **Telegram: Send Response**

9. **DataStore: Log**

**Output:**
```
📧 **5 מיילים לא נקראים:**

1. [🔴 דחוף 5/5] מג'ון - חשבונית לתשלום
   📌 פעולה: שלח אישור עד מחר
...
```

---

### WF4: Get Email by ID

**Purpose:** Read and analyze a specific email

**Trigger:** Internal call (intent: read)

**Input:** `email_id`

**Nodes:**

1. **Gmail: Get Email**

2. **Code: Clean Body**
   - Strip HTML
   - Strip quoted text

3. **OpenAI: Analyze Email**
   - Extract: sentiment, urgency, category, summary, action items

4. **Code: Format Response**
   - Build rich message with insights

5. **Telegram: Send**

6. **DataStore: Log**

---

### WF5: Summarize Thread

**Purpose:** Summarize an entire email conversation

**Trigger:** Internal call (intent: summarize, thread_id)

**Input:** `thread_id`

**Nodes:**

1. **Gmail: Get Thread**

2. **Code: Extract Messages**
   - Sort by date
   - For each: strip quoted text

3. **OpenAI: Summarize Thread**
   - System: "Summarize this email thread in 2-3 sentences. Include action items and next step."
   - Input: all messages

4. **Code: Format Response**

5. **Telegram: Send**

6. **DataStore: Log**

---

### WF6: Draft Reply

**Purpose:** Generate and preview email draft

**Trigger:** Internal call (intent: draft_reply)

**Input:** `thread_id` or `email_id`, user instructions

**Nodes:**

1. **Gmail: Get Email/Thread**
   - Fetch context

2. **Code: Extract Last Message**
   - Get most recent message for reply context

3. **OpenAI: Generate Draft**
   - System: "Write a professional email reply. Structure: greeting, response, question/CTA, signature placeholder."
   - Input: last message + user instructions
   - Output: draft body

4. **Code: Build Preview Message**
   - Show draft in formatted block
   - Add confirmation buttons

5. **Telegram: Send Preview**

6. **DataStore: Save Pending Action**
   ```json
   {
     "user_id": "...",
     "action_type": "create_draft",
     "params": {
       "thread_id": "...",
       "to": "...",
       "subject": "Re: ...",
       "body": "[draft text]"
     },
     "created_at": "...",
     "expires_at": "[15 min from now]"
   }
   ```

7. **END** (wait for confirmation via WF8)

---

### WF7: Bulk Label & Archive

**Purpose:** Label and archive multiple emails with dry run

**Trigger:** Internal call (intent: label or archive, bulk)

**Input:** `query`, `labelIds`, `archive: true/false`

**Nodes:**

1. **Gmail: Search Emails**
   - Use query
   - Limit: 100

2. **Code: Build Dry Run Preview**
   - Count total
   - Get first 3-5 as examples
   - Format message

3. **Telegram: Send Dry Run**

4. **DataStore: Save Pending Action**
   ```json
   {
     "user_id": "...",
     "action_type": "bulk_label_archive",
     "params": {
       "query": "...",
       "labelIds": ["..."],
       "archive": true,
       "dry_run_count": 47,
       "sample_ids": ["id1", "id2", "id3"]
     },
     "created_at": "..."
   }
   ```

5. **END** (wait for confirmation)

---

### WF8: Confirmation Handler

**Purpose:** Execute pending actions after user confirmation

**Trigger:** Telegram message = "אשר" or callback button

**Nodes:**

1. **Telegram: Receive Confirmation**

2. **DataStore: Load Pending Action**
   - Query by user_id
   - Check expiration (< 15 min old)

3. **IF: No Pending Action**
   - Reply: "אין פעולה ממתינה"
   - END

4. **Code: Re-validate Params**
   - Ensure still safe to execute

5. **Switch: Route by action_type**
   - `create_draft` → Execute Draft Creation
   - `bulk_label_archive` → Execute Bulk Operations
   - `mark_read` → Execute Mark Read
   - etc.

6. **FOR action_type = bulk_label_archive:**

   a. **Loop: For Each Email ID**
   
   b. **Code: Check Idempotency**
      - Generate key: `hash(user_id + 'label_archive' + email_id)`
      - Query idempotency_keys table
      - IF exists → skip this email
   
   c. **Gmail: Modify Labels**
      - addLabelIds: [labelIds from params]
      - removeLabelIds: ["INBOX"] if archive
   
   d. **DataStore: Save Idempotency Key**
      - key, user_id, action_type, timestamp
   
   e. **DataStore: Log Action**
      - Audit log entry

7. **Code: Build Result Summary**
   - Count success, failures
   - Format message

8. **Telegram: Send Result**

9. **DataStore: Clear Pending Action**
   - DELETE from pending_actions WHERE user_id = ?

10. **DataStore: Log Completion**

**Output:**
```
✅ **הפעולה הושלמה!**

- 47 מיילים תויגו כ-Newsletter
- 47 מיילים אורכבו
- 0 כשלונות
```

---

## 💾 Data Layer

### Database Tables

#### 1. session_state
```sql
CREATE TABLE session_state (
  user_id VARCHAR(255) PRIMARY KEY,
  last_intent VARCHAR(100),
  last_email_ids TEXT[], -- Array of email IDs from last search
  last_context JSONB, -- Free-form context for conversation
  preferences JSONB DEFAULT '{
    "default_limit": 10,
    "redaction_enabled": true,
    "language": "he"
  }',
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Index for fast lookup
CREATE INDEX idx_session_user ON session_state(user_id);

-- TTL: Auto-delete after 24 hours
-- (Implementation depends on DB: Postgres cron, n8n scheduled workflow, etc.)
```

**Purpose:** Store short-term conversation state per user

**Example:**
```json
{
  "user_id": "telegram_123456",
  "last_intent": "search",
  "last_email_ids": ["abc123", "def456", "ghi789"],
  "last_context": {
    "last_query": "from:john",
    "last_thread_id": "thread_xyz"
  },
  "preferences": {
    "default_limit": 10,
    "redaction_enabled": true,
    "language": "he"
  },
  "updated_at": "2026-01-29T10:30:00Z"
}
```

---

#### 2. pending_actions
```sql
CREATE TABLE pending_actions (
  user_id VARCHAR(255) PRIMARY KEY,
  action_type VARCHAR(100) NOT NULL,
  params JSONB NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  expires_at TIMESTAMP NOT NULL
);

-- Index
CREATE INDEX idx_pending_user ON pending_actions(user_id);
CREATE INDEX idx_pending_expires ON pending_actions(expires_at);

-- TTL: Auto-delete expired actions
```

**Purpose:** Store actions awaiting user confirmation

**Example:**
```json
{
  "user_id": "telegram_123456",
  "action_type": "create_draft",
  "params": {
    "thread_id": "thread_xyz",
    "to": "john@example.com",
    "subject": "Re: Invoice",
    "body": "Hi John,\n\nThank you for the invoice..."
  },
  "created_at": "2026-01-29T10:30:00Z",
  "expires_at": "2026-01-29T10:45:00Z"
}
```

---

#### 3. idempotency_keys
```sql
CREATE TABLE idempotency_keys (
  key VARCHAR(255) PRIMARY KEY,
  user_id VARCHAR(255) NOT NULL,
  action_type VARCHAR(100) NOT NULL,
  email_id VARCHAR(255),
  executed_at TIMESTAMP DEFAULT NOW(),
  result JSONB
);

-- Indexes
CREATE INDEX idx_idem_user ON idempotency_keys(user_id);
CREATE INDEX idx_idem_action ON idempotency_keys(action_type);

-- TTL: Auto-delete after 7 days
```

**Purpose:** Prevent duplicate actions on retries

**Key Generation:**
```javascript
const crypto = require('crypto');

function generateIdempotencyKey(userId, actionType, emailId, params) {
  const data = `${userId}_${actionType}_${emailId}_${JSON.stringify(params)}`;
  return crypto.createHash('sha256').update(data).digest('hex');
}
```

**Example:**
```json
{
  "key": "a3f5d2c8b1e9f4a7c6d8e2b5f3a9c7d4e8b2f5a3c6d9e7b4f8a2c5d3e6b9f7a4",
  "user_id": "telegram_123456",
  "action_type": "label_archive",
  "email_id": "abc123",
  "executed_at": "2026-01-29T10:30:00Z",
  "result": {
    "success": true,
    "labels_added": ["Newsletter"],
    "archived": true
  }
}
```

**Usage in Workflow:**
```javascript
// Before executing action
const key = generateIdempotencyKey(userId, 'label_archive', emailId, params);
const existing = await db.query('SELECT * FROM idempotency_keys WHERE key = ?', [key]);

if (existing) {
  // Already executed, return cached result
  return existing.result;
}

// Execute action
const result = await gmailApi.modifyLabels(emailId, params);

// Save idempotency key
await db.insert('idempotency_keys', {
  key,
  user_id: userId,
  action_type: 'label_archive',
  email_id: emailId,
  result
});

return result;
```

---

#### 4. audit_log
```sql
CREATE TABLE audit_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  timestamp TIMESTAMP DEFAULT NOW(),
  user_id VARCHAR(255) NOT NULL,
  intent VARCHAR(100),
  action_type VARCHAR(100),
  params_summary JSONB, -- Minimal params, NO full email bodies
  execution_confirmed BOOLEAN DEFAULT false,
  success BOOLEAN,
  error_code VARCHAR(50),
  error_message TEXT
);

-- Indexes
CREATE INDEX idx_audit_user ON audit_log(user_id);
CREATE INDEX idx_audit_timestamp ON audit_log(timestamp DESC);
CREATE INDEX idx_audit_action ON audit_log(action_type);
```

**Purpose:** Complete audit trail for all actions

**Example:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "timestamp": "2026-01-29T10:30:00Z",
  "user_id": "telegram_123456",
  "intent": "label",
  "action_type": "bulk_label_archive",
  "params_summary": {
    "query": "from:newsletter",
    "labelIds": ["Newsletter"],
    "count": 47
  },
  "execution_confirmed": true,
  "success": true,
  "error_code": null,
  "error_message": null
}
```

**Privacy Note:** 
❌ Do NOT store: full email bodies, sensitive content  
✅ DO store: email IDs, counts, query strings, action types

---

## 🔒 Security & Privacy

### 1. Content Cleaning

#### HTML Stripping
```javascript
// Node: Code - Clean HTML
function cleanHTML(html) {
  if (!html) return '';
  
  // Remove HTML tags
  let text = html.replace(/<style[^>]*>.*?<\/style>/gis, '');
  let text = text.replace(/<script[^>]*>.*?<\/script>/gis, '');
  let text = text.replace(/<[^>]+>/g, '');
  
  // Decode HTML entities
  text = text.replace(/&nbsp;/g, ' ');
  text = text.replace(/&amp;/g, '&');
  text = text.replace(/&lt;/g, '<');
  text = text.replace(/&gt;/g, '>');
  text = text.replace(/&quot;/g, '"');
  
  // Remove excessive whitespace
  text = text.replace(/\s+/g, ' ').trim();
  
  return text;
}

// Usage in workflow
const cleanBody = cleanHTML(items[0].json.body.html);
```

#### Quoted History Removal
```javascript
// Node: Code - Strip Quoted Text
function stripQuotedHistory(body) {
  if (!body) return '';
  
  // Remove lines starting with >
  let lines = body.split('\n');
  lines = lines.filter(line => {
    const trimmed = line.trim();
    return !trimmed.startsWith('>') && !trimmed.startsWith('&gt;');
  });
  
  // Remove "On [date], [name] wrote:" blocks
  let text = lines.join('\n');
  text = text.replace(/On .+? wrote:/gi, '');
  text = text.replace(/ב.+? כתב:/gi, ''); // Hebrew
  
  // Remove common email client quoted text markers
  text = text.replace(/From: .+?Sent: .+?To: .+?Subject:.+?\n/gis, '');
  
  return text.trim();
}
```

---

### 2. PII Redaction

```javascript
// Node: Code - Redact PII
function redactPII(text, options = {}) {
  if (!text) return '';
  
  let redacted = text;
  
  // Phone numbers (Israeli format)
  if (options.redactPhone !== false) {
    // 050-1234567, 0501234567, +972-50-1234567
    redacted = redacted.replace(/(\+972-?|0)5[0-9]-?\d{7}/g, '[PHONE]');
    redacted = redacted.replace(/(\+972-?|0)[2-9]-?\d{7}/g, '[PHONE]');
  }
  
  // Israeli ID numbers (9 digits)
  if (options.redactID !== false) {
    redacted = redacted.replace(/\b\d{9}\b/g, '[ID]');
  }
  
  // Credit card numbers (basic pattern)
  if (options.redactCreditCard !== false) {
    redacted = redacted.replace(/\b\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}\b/g, '[CARD]');
  }
  
  // Email addresses (optional)
  if (options.redactEmail === true) {
    redacted = redacted.replace(/[\w.-]+@[\w.-]+\.\w+/g, '[EMAIL]');
  }
  
  return redacted;
}

// Usage in workflow
const userPrefs = session.preferences || {};
if (userPrefs.redaction_enabled) {
  const cleanText = redactPII(bodyText, {
    redactPhone: true,
    redactID: true,
    redactCreditCard: true,
    redactEmail: false // Keep emails by default
  });
}
```

---

### 3. Minimal Data to AI

**Policy:** Only send to OpenAI:
- `subject` (max 200 chars)
- `from` (email address)
- `date` (ISO format)
- `snippet` (max 500 chars, cleaned & redacted)

**Implementation:**
```javascript
// Node: Code - Filter for AI
function prepareForAI(email, userPrefs) {
  // Start with minimal data
  let aiInput = {
    subject: email.subject?.substring(0, 200) || '(no subject)',
    from: email.from,
    date: email.date
  };
  
  // Only include body if specifically needed
  if (email.needsFullBody) {
    let body = email.bodyText || email.bodyHtml;
    
    // Clean
    if (email.bodyHtml) {
      body = cleanHTML(body);
    }
    
    // Strip quoted
    body = stripQuotedHistory(body);
    
    // Redact if enabled
    if (userPrefs.redaction_enabled) {
      body = redactPII(body);
    }
    
    // Truncate
    aiInput.snippet = body.substring(0, 500);
    if (body.length > 500) {
      aiInput.snippet += '... [truncated]';
    }
  } else {
    // Use Gmail snippet
    aiInput.snippet = email.snippet || '';
  }
  
  return aiInput;
}
```

---

### 4. Confirmation Gate Enforcement

```javascript
// Node: Code - Requires Confirmation Check
function requiresConfirmation(decision) {
  const { action, intent } = decision;
  
  // Destructive actions always need confirmation
  const destructiveActions = [
    'gmail.archive',
    'gmail.mark_read',
    'gmail.label',
    'gmail.draft' // Even drafts need preview
  ];
  
  if (destructiveActions.includes(action.type)) {
    return true;
  }
  
  // Bulk operations need confirmation
  const isBulk = (
    action.params.query || // Search query = potential bulk
    (action.params.email_ids && action.params.email_ids.length > 1)
  );
  
  if (isBulk) {
    return true;
  }
  
  // Respect AI decision
  if (decision.needs_confirmation === true) {
    return true;
  }
  
  return false;
}

// Usage in workflow
const needsConfirm = requiresConfirmation(decision);

if (needsConfirm && !userConfirmed) {
  // BLOCK execution
  // Build preview
  // Request confirmation
  // END workflow
  return {
    blocked: true,
    reason: 'confirmation_required'
  };
}
```

---

### 5. OAuth Scopes (Least Privilege)

**Required Scopes:**
```json
{
  "scopes": [
    "https://www.googleapis.com/auth/gmail.readonly",
    "https://www.googleapis.com/auth/gmail.modify",
    "https://www.googleapis.com/auth/gmail.compose"
  ]
}
```

**Explicitly Forbidden:**
```json
{
  "forbidden_scopes": [
    "https://www.googleapis.com/auth/gmail.send"
  ]
}
```

**Testing:**
```bash
# Test that sending is blocked
curl -X POST \
  'https://gmail.googleapis.com/gmail/v1/users/me/messages/send' \
  -H 'Authorization: Bearer [token]' \
  -d '{"raw": "[base64_email]"}'

# Expected: 403 Forbidden (insufficient permissions)
```

---

### 6. No Body Storage by Default

**Session State Policy:**
```javascript
// DO NOT store full bodies in session
function saveToSession(userId, data) {
  const sessionData = {
    user_id: userId,
    last_intent: data.intent,
    last_email_ids: data.email_ids, // IDs only, not bodies
    last_context: {
      last_query: data.query,
      last_thread_id: data.thread_id
      // NO: last_body, last_content
    },
    updated_at: new Date().toISOString()
  };
  
  return db.upsert('session_state', sessionData);
}
```

**Audit Log Policy:**
```javascript
// Only store minimal params
function auditLog(userId, action, params, result) {
  const logEntry = {
    user_id: userId,
    action_type: action.type,
    params_summary: {
      email_id: params.email_id,
      query: params.query,
      count: params.email_ids?.length || 1
      // NO: body, content, full_text
    },
    success: result.success,
    timestamp: new Date().toISOString()
  };
  
  return db.insert('audit_log', logEntry);
}
```

**User Preference Toggle:**
```json
{
  "preferences": {
    "save_bodies": false, // Default: do not save
    "redaction_enabled": true,
    "verbose_logging": false
  }
}
```

---

## 🧪 Testing Plan

### Test Matrix

| # | Test Name | Type | Priority | Estimated Time |
|---|-----------|------|----------|----------------|
| 1 | Summarize Unread (Happy Path) | Functional | High | 15 min |
| 2 | Search with Query | Functional | High | 10 min |
| 3 | Read Email by ID | Functional | High | 10 min |
| 4 | Summarize Thread | Functional | High | 15 min |
| 5 | Draft Reply with Confirmation | Functional | High | 20 min |
| 6 | Bulk Label & Archive | Functional | High | 25 min |
| 7 | Invalid AI JSON | Error Handling | High | 15 min |
| 8 | Rate Limit Retry | Error Handling | Medium | 20 min |
| 9 | HTML Cleaning | Data Processing | Medium | 15 min |
| 10 | Block No-Confirm Destructive | Security | High | 10 min |
| 11 | PII Redaction | Privacy | Medium | 15 min |
| 12 | Idempotency Check | Reliability | High | 20 min |
| 13 | Session Persistence | State Management | Medium | 10 min |
| 14 | Pending Action Expiration | State Management | Low | 10 min |
| 15 | Audit Log Completeness | Compliance | Medium | 15 min |

**Total Estimated Testing Time: ~4 hours**

---

### Test Execution Checklist

#### Before Testing:
- [ ] All workflows imported and activated
- [ ] Test Gmail account prepared with sample emails
- [ ] Telegram bot connected
- [ ] Database tables created
- [ ] n8n execution logs enabled
- [ ] Screen recording tool ready

#### For Each Test:
- [ ] Document test input
- [ ] Execute test
- [ ] Capture execution log
- [ ] Take screenshot of result
- [ ] Verify expected outcome
- [ ] Document any deviations
- [ ] Save evidence (log + screenshot)

#### After Testing:
- [ ] Fill results table
- [ ] Identify any bugs or issues
- [ ] Prioritize fixes
- [ ] Re-test after fixes
- [ ] Mark test as PASS/FAIL

---

### Evidence Collection

**Folder Structure:**
```
/test-evidence
  ├── test-001-summarize-unread/
  │   ├── input.txt
  │   ├── execution-log.json
  │   ├── telegram-screenshot.png
  │   └── result.md
  ├── test-002-search-query/
  │   ├── input.txt
  │   ├── execution-log.json
  │   └── result.md
  ...
```

**Log Export:**
- From n8n: Execution → Click execution → Export JSON
- Save as `execution-log.json`

**Screenshot:**
- Telegram conversation showing:
  - User input
  - Bot response
  - Confirmation flow (if applicable)

**Result Document:**
```markdown
# Test 001: Summarize Unread

## Input
"מה דחוף היום"

## Expected
- AI decision: summarize, confidence > 0.70
- Gmail search: is:unread, limit 10
- Response: list of summaries with urgency

## Actual
✅ AI decision: summarize, confidence: 0.85
✅ Gmail search executed: 5 results
✅ Response formatted correctly with urgency indicators

## Result: PASS

## Evidence
- execution-log.json
- telegram-screenshot.png

## Notes
No issues found. Email with only HTML body was cleaned correctly.
```

---

## 🚀 Installation Guide

### Prerequisites

- **Docker & Docker Compose** (recommended) or Node.js 18+
- **Telegram Account** (for bot)
- **OpenAI API Key**
- **Google Cloud Project** (for Gmail API)
- **PostgreSQL** (optional, can use n8n DataStore)

---

### Step 1: Setup n8n

#### Option A: Docker Compose (Recommended)

Create `docker-compose.yml`:
```yaml
version: '3.8'

services:
  n8n:
    image: n8nio/n8n:latest
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_HOST=${N8N_HOST}
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - NODE_ENV=production
      - WEBHOOK_URL=${WEBHOOK_URL}
      - N8N_ENCRYPTION_KEY=${N8N_ENCRYPTION_KEY}
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=${POSTGRES_DB}
      - DB_POSTGRESDB_USER=${POSTGRES_USER}
      - DB_POSTGRESDB_PASSWORD=${POSTGRES_PASSWORD}
    volumes:
      - n8n_data:/home/node/.n8n
    depends_on:
      - postgres

  postgres:
    image: postgres:15
    restart: always
    environment:
      - POSTGRES_DB=${POSTGRES_DB}
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  n8n_data:
  postgres_data:
```

Create `.env`:
```bash
N8N_HOST=localhost
N8N_PORT=5678
WEBHOOK_URL=http://localhost:5678/
N8N_ENCRYPTION_KEY=your-very-secure-random-encryption-key-at-least-10-chars

POSTGRES_DB=n8n
POSTGRES_USER=n8n
POSTGRES_PASSWORD=your-secure-db-password
```

Start:
```bash
docker-compose up -d
```

#### Option B: npm
```bash
npm install n8n -g
n8n start
```

Access n8n: http://localhost:5678

---

### Step 2: Create Telegram Bot

1. Open Telegram, search for **@BotFather**
2. Send `/newbot`
3. Follow instructions (name, username)
4. Copy the **bot token** (looks like `123456789:ABCdefGHIjklMNOpqrsTUVwxyz`)
5. Save token for later

---

### Step 3: Setup OpenAI

1. Go to https://platform.openai.com/api-keys
2. Create new API key
3. Copy and save securely

---

### Step 4: Setup Gmail API

1. **Create Google Cloud Project:**
   - Go to https://console.cloud.google.com
   - Create new project: "AI Email Copilot"

2. **Enable Gmail API:**
   - Navigate to "APIs & Services" → "Library"
   - Search "Gmail API"
   - Click "Enable"

3. **Configure OAuth Consent Screen:**
   - Go to "APIs & Services" → "OAuth consent screen"
   - Select "External" (for testing) or "Internal" (for organization)
   - Fill required fields:
     - App name: "AI Email Copilot"
     - User support email: your email
     - Developer contact: your email
   - Add scopes:
     - `https://www.googleapis.com/auth/gmail.readonly`
     - `https://www.googleapis.com/auth/gmail.modify`
     - `https://www.googleapis.com/auth/gmail.compose`
   - Save

4. **Create OAuth Credentials:**
   - Go to "APIs & Services" → "Credentials"
   - Click "Create Credentials" → "OAuth client ID"
   - Application type: "Web application"
   - Name: "n8n OAuth"
   - Authorized redirect URIs:
     - `http://localhost:5678/rest/oauth2-credential/callback`
     - (Add your production URL if deploying)
   - Click "Create"
   - **Copy Client ID and Client Secret**

---

### Step 5: Configure n8n Credentials

1. **Open n8n** (http://localhost:5678)

2. **Add OpenAI Credential:**
   - Go to "Credentials" → "New"
   - Type: "OpenAI"
   - API Key: [paste your OpenAI key]
   - Save

3. **Add Gmail OAuth2 Credential:**
   - Go to "Credentials" → "New"
   - Type: "Gmail OAuth2"
   - Client ID: [from Google Cloud]
   - Client Secret: [from Google Cloud]
   - Click "Connect my account"
   - Authorize with your Gmail
   - Save

4. **Add Telegram Credential:**
   - Type: "Telegram"
   - Bot Token: [from BotFather]
   - Save

---

### Step 6: Import Workflows

1. **Download workflow JSON files** (from `/workflows` folder)

2. **Import each workflow:**
   - In n8n: Click "+" → "Import from File"
   - Select JSON file
   - Review nodes
   - Update credentials references to your saved credentials
   - Save workflow

3. **Workflows to import (in order):**
   - WF8_confirmation_handler.json (first, as others may call it)
   - WF2_gmail_search.json
   - WF3_summarize_unread.json
   - WF4_get_email.json
   - WF5_summarize_thread.json
   - WF6_draft_reply.json
   - WF7_bulk_label_archive.json
   - WF1_chat_ingest_decide.json (main entry point)

4. **Activate all workflows**

---

### Step 7: Setup Database Tables

If using PostgreSQL, run this SQL:

```sql
-- Session State
CREATE TABLE session_state (
  user_id VARCHAR(255) PRIMARY KEY,
  last_intent VARCHAR(100),
  last_email_ids TEXT[],
  last_context JSONB,
  preferences JSONB DEFAULT '{"default_limit": 10, "redaction_enabled": true, "language": "he"}',
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Pending Actions
CREATE TABLE pending_actions (
  user_id VARCHAR(255) PRIMARY KEY,
  action_type VARCHAR(100) NOT NULL,
  params JSONB NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  expires_at TIMESTAMP NOT NULL
);

-- Idempotency Keys
CREATE TABLE idempotency_keys (
  key VARCHAR(255) PRIMARY KEY,
  user_id VARCHAR(255) NOT NULL,
  action_type VARCHAR(100) NOT NULL,
  email_id VARCHAR(255),
  executed_at TIMESTAMP DEFAULT NOW(),
  result JSONB
);

-- Audit Log
CREATE TABLE audit_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  timestamp TIMESTAMP DEFAULT NOW(),
  user_id VARCHAR(255) NOT NULL,
  intent VARCHAR(100),
  action_type VARCHAR(100),
  params_summary JSONB,
  execution_confirmed BOOLEAN DEFAULT false,
  success BOOLEAN,
  error_code VARCHAR(50),
  error_message TEXT
);

-- Indexes
CREATE INDEX idx_session_user ON session_state(user_id);
CREATE INDEX idx_pending_user ON pending_actions(user_id);
CREATE INDEX idx_idem_user ON idempotency_keys(user_id);
CREATE INDEX idx_audit_user ON audit_log(user_id);
CREATE INDEX idx_audit_timestamp ON audit_log(timestamp DESC);
```

If using n8n DataStore, tables are managed automatically.

---

### Step 8: Test Basic Connection

1. **Open Telegram**
2. **Find your bot** (search by username)
3. **Send:** `/start`
4. **Expected:** Bot responds with welcome message

If no response:
- Check n8n workflow "WF1_chat_ingest_decide" is activated
- Check execution log in n8n for errors
- Verify Telegram credential is correct

---

### Step 9: Connect Gmail Account

1. **In Telegram, send:** "connect gmail" (if you built this feature)
   OR
2. **Gmail OAuth already connected in step 5** - skip this

3. **Test:** Send "מה דחוף היום"
4. **Expected:** Bot searches your Gmail and returns unread summaries

---

### Step 10: Run Tests

Follow **Testing Plan** section to verify all functionality.

---

## 📚 Project Backlog

### Epic 1: Foundation ✅
- [x] Setup n8n environment
- [x] Configure Telegram bot
- [x] Setup OpenAI credentials
- [x] Setup Gmail OAuth
- [x] Test end-to-end connection

### Epic 2: Gmail Integration ✅
- [x] Gmail Search node
- [x] Gmail Get Email node
- [x] Gmail Get Thread node
- [x] Gmail Create Draft node
- [x] Gmail Modify Labels node
- [x] Error handling & retries

### Epic 3: AI Agent ✅
- [x] System prompt with guardrails
- [x] JSON schema definition
- [x] Validation node
- [x] Confidence threshold logic
- [x] Confirmation requirement rules
- [x] Intent detection tests

### Epic 4: Core Workflows ✅
- [x] WF1: Chat Ingest & Decide
- [x] WF2: Gmail Search
- [x] WF3: Summarize Unread
- [x] WF4: Get Email
- [x] WF5: Summarize Thread
- [x] WF6: Draft Reply
- [x] WF7: Bulk Operations
- [x] WF8: Confirmation Handler

### Epic 5: Data Layer ✅
- [x] Session state table
- [x] Pending actions table
- [x] Idempotency keys table
- [x] Audit log table

### Epic 6: Security & Privacy ✅
- [x] HTML cleaning
- [x] Quoted text removal
- [x] PII redaction
- [x] Minimal data to AI
- [x] Confirmation gates
- [x] Least privilege scopes
- [x] No body storage by default

### Epic 7: Testing ✅
- [x] Test plan with 10+ scenarios
- [x] Evidence collection
- [x] Results documentation

### Epic 8: Documentation ✅
- [x] README.md
- [x] ARCHITECTURE.md
- [x] Installation guide
- [x] Workflow exports

---

### Future Enhancements (Post-MVP)

#### Phase 9: Advanced Features
- [ ] **Web UI** instead of Telegram
  - Dashboard view of inbox
  - Visual action approval
  - History view
- [ ] **Multi-account support**
  - Switch between Gmail accounts
  - Unified search across accounts
- [ ] **Vector search** for semantic email search
  - Store email embeddings
  - "Find emails about X" without exact keywords
- [ ] **Smart categorization**
  - Auto-detect newsletters, VIPs, work vs personal
  - Learn from user behavior
- [ ] **Scheduled actions**
  - "Remind me about this email tomorrow"
  - "Archive all newsletters every Friday"
- [ ] **Attachment handling**
  - Summarize PDFs
  - Extract text from images
  - Download and process attachments

#### Phase 10: Intelligence
- [ ] **Learning user preferences**
  - Remember draft style
  - Learn urgency scoring from feedback
  - Personalize categories
- [ ] **Proactive suggestions**
  - "You haven't replied to John in 3 days"
  - "This looks like a bill, want to mark for payment?"
- [ ] **Email templates**
  - Save common reply patterns
  - Quick reply suggestions based on context

#### Phase 11: Integrations
- [ ] **Calendar integration**
  - "Schedule meeting from this email"
  - Auto-detect meeting requests
- [ ] **Task management**
  - Create tasks from action items
  - Integration with Todoist, Notion, etc.
- [ ] **CRM integration**
  - Auto-log emails to CRM
  - Show contact context

#### Phase 12: Scale & Performance
- [ ] **Redis for caching**
  - Cache frequent searches
  - Session state in Redis
- [ ] **Background job queue**
  - Async processing for bulk operations
  - Retry queue for failed actions
- [ ] **Metrics & monitoring**
  - Grafana dashboards
  - Error rate alerts
  - Usage analytics

---

## 💡 Development Guidelines

### Working with Cursor AI

**What to generate with Cursor:**

1. **Utility Functions:**
```bash
# Prompt for Cursor:
"Generate a Node.js Gmail client wrapper with methods: searchMessages, getMessage, getThread, createDraft, modifyLabels, archive, markRead. Include OAuth token refresh, retry with exponential backoff on 429 and 5xx, and TypeScript types."
```

2. **JSON Schema & Validation:**
```bash
# Prompt:
"Create a JSON Schema for the AI decision contract with fields: intent, confidence, needs_confirmation, message_to_user, email_insights, action. Add Ajv validation with examples and error messages in Hebrew."
```

3. **Test Suites:**
```bash
# Prompt:
"Write Jest unit tests for Gmail wrapper. Mock Gmail API responses. Cover: rate limit retry, token refresh, idempotency key generation, error handling. Include test data fixtures."
```

4. **Docker Setup:**
```bash
# Prompt:
"Create Docker Compose for n8n + Postgres. Include .env.example with all required variables, health checks, volume persistence, and security best practices. Add README for setup."
```

5. **Data Processing:**
```bash
# Prompt:
"Generate utility functions: stripQuotedText(emailBody), htmlToTextClean(html), redactPII(text) with Israeli phone/ID patterns, Base64 encoding/decoding for Gmail API. Add TypeScript types and unit tests."
```

---

### Division of Labor: You vs. Cursor

**You handle:**
- Architecture decisions
- n8n workflow design
- AI prompt engineering
- Business logic & guardrails
- Testing scenarios
- Security policies

**Cursor handles:**
- Boilerplate code
- Type definitions
- Unit tests
- Docker configs
- Utility functions
- Code documentation

---

### Best Practices

1. **One workflow, one purpose**
   - Don't create mega-workflows
   - Keep workflows focused and testable

2. **Always validate AI output**
   - Never trust AI JSON without validation
   - Block execution on invalid schema

3. **Log everything**
   - Every action attempt (success or failure)
   - Include user_id, timestamp, params summary
   - Never log full email bodies

4. **Idempotency first**
   - Generate key before execution
   - Check before acting
   - Cache results

5. **Fail gracefully**
   - Catch all errors
   - Friendly user messages
   - Detailed logs for debugging

6. **Test on real data**
   - Use actual Gmail account with varied emails
   - Test HTML-heavy emails
   - Test long threads
   - Test edge cases (no subject, empty body, etc.)

---

### Debugging Tips

**If AI returns invalid JSON:**
1. Check system prompt is correctly set
2. Verify OpenAI node is set to JSON mode
3. Look at raw response in execution log
4. Adjust prompt to be more explicit

**If Gmail API fails:**
1. Check OAuth token hasn't expired
2. Verify scopes are correct
3. Check rate limits (429 error)
4. Look for quota exceeded errors

**If confirmation doesn't work:**
1. Check pending_action was saved
2. Verify user_id matches
3. Check expiration time
4. Look at execution log for WF8

**If duplicate actions:**
1. Check idempotency key generation
2. Verify key is checked before execution
3. Look at idempotency_keys table

---

### Environment Variables Reference

```bash
# n8n Core
N8N_HOST=localhost
N8N_PORT=5678
N8N_PROTOCOL=http
N8N_ENCRYPTION_KEY=your-secure-key-min-10-chars
WEBHOOK_URL=http://localhost:5678/

# Database
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=postgres
DB_POSTGRESDB_PORT=5432
DB_POSTGRESDB_DATABASE=n8n
DB_POSTGRESDB_USER=n8n
DB_POSTGRESDB_PASSWORD=your-secure-password

# Or use DataStore (no DB vars needed)
# DB_TYPE=datastore

# Telegram (saved in n8n credentials, not env)
# TELEGRAM_BOT_TOKEN=123456789:ABCdefGHIjklMNOpqrsTUVwxyz

# OpenAI (saved in n8n credentials, not env)
# OPENAI_API_KEY=sk-...

# Gmail OAuth (saved in n8n credentials, not env)
# GMAIL_CLIENT_ID=...
# GMAIL_CLIENT_SECRET=...
```

---

## 🎓 Learning Resources

### n8n
- [Official Docs](https://docs.n8n.io)
- [Community Forum](https://community.n8n.io)
- [YouTube Tutorials](https://www.youtube.com/c/n8n-io)

### Gmail API
- [REST Reference](https://developers.google.com/gmail/api/reference/rest)
- [Query Syntax](https://support.google.com/mail/answer/7190?hl=en)
- [Best Practices](https://developers.google.com/gmail/api/guides/best-practices)

### OpenAI
- [API Docs](https://platform.openai.com/docs)
- [JSON Mode](https://platform.openai.com/docs/guides/text-generation/json-mode)
- [Best Practices for Prompts](https://platform.openai.com/docs/guides/prompt-engineering)

### Telegram Bots
- [Bot API](https://core.telegram.org/bots/api)
- [BotFather Commands](https://core.telegram.org/bots#botfather)

---

## 🤝 Contributing

*(If open source)*

### How to Contribute

1. **Fork** the repository
2. **Create a branch** for your feature
3. **Make changes** and test thoroughly
4. **Submit a pull request** with clear description

### Contribution Guidelines

- Follow existing code style
- Add tests for new features
- Update documentation
- Keep workflows modular and focused

---

## 📄 License

*(Choose appropriate license)*

**Options:**
- **MIT License**: Open source, permissive
- **Apache 2.0**: Open source, includes patent grant
- **Proprietary**: Closed source, all rights reserved

---

## 🙏 Acknowledgments

- **n8n**: Workflow automation platform
- **OpenAI**: GPT models for AI agent
- **Gmail API**: Email integration
- **Telegram**: Chat interface
- **Community contributors**: (if applicable)

---

## 📞 Support & Contact

**Issues:** [GitHub Issues](link) or support email

**Documentation:** This README + inline workflow comments

**Updates:** Check [CHANGELOG.md](./CHANGELOG.md) for version history

---

**Version:** 1.0.0 (MVP)  
**Last Updated:** 2026-01-29  
**Author:** [Your Name/Team]

---

## 🎯 Quick Start Summary

```bash
# 1. Clone repo
git clone [repo-url]
cd ai-email-copilot

# 2. Setup environment
cp .env.example .env
# Edit .env with your values

# 3. Start n8n
docker-compose up -d

# 4. Access n8n
open http://localhost:5678

# 5. Import workflows
# (via n8n UI)

# 6. Configure credentials
# OpenAI, Gmail OAuth, Telegram

# 7. Activate workflows

# 8. Test
# Send "מה דחוף היום" to your Telegram bot
```

---

**🚀 You're ready to build! בהצלחה!**