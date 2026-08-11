# Resilient AI Lead Router

An n8n workflow that receives inbound lead submissions via a webhook, normalizes the
data, scores each lead with Google Gemini, validates the AI response, and routes the
lead to a Google Sheet based on the AI's verdict.

## Flow overview

```
Webhook → Clean Data → AI Scoring → Validate AI → Route Leads → Append row in sheet
```

---

## Nodes

### 1. Webhook
**Type:** `n8n-nodes-base.webhook` · **Trigger**

- **Input:** An external HTTP `POST` request to the webhook path. The request body is
  expected to contain nested lead information, e.g.:
  ```json
  {
    "user_info":  { "full_name": "jane doe", "email_address": "JANE@ACME.COM" },
    "form_data":  { "company_name": "Acme Inc", "message_text": "Interested in a demo" },
    "metadata":   { "source": "website" }
  }
  ```
- **Output:** One item whose `json.body` holds the raw request payload, plus HTTP
  metadata (headers, query params).

---

### 2. Clean Data
**Type:** `n8n-nodes-base.code` (JavaScript)

- **Input:** The raw webhook item. Reads from `item.json.body` (falls back to
  `item.json`), pulling `user_info.full_name`, `user_info.email_address`,
  `form_data.company_name`, `form_data.message_text`, and `metadata.source`.
- **Processing:** Trims and title-cases the name, lowercases the email, derives the
  email `domain`, and infers a `company` from the company field or the domain.
- **Output:** A normalized lead object:
  ```json
  {
    "name":    "Jane Doe",
    "email":   "jane@acme.com",
    "domain":  "acme.com",
    "company": "Acme",
    "message": "Interested in a demo",
    "source":  "website"
  }
  ```

---

### 3. AI Scoring
**Type:** `n8n-nodes-base.httpRequest` → Google Gemini `generateContent`

- **Input:** The cleaned lead object. Injects `name`, `company`, `domain`, and
  `message` into a B2B qualification prompt via expressions.
- **Processing:** Sends a `POST` request to the Gemini API asking for a strict JSON
  verdict (`temperature: 0.1`, `responseMimeType: application/json`).
- **Output:** The raw Gemini API response. The verdict text sits at
  `candidates[0].content.parts[0].text` and is expected to be JSON of the form:
  ```json
  { "score": 8, "reasoning": "Strong SaaS fit", "action": "hot" }
  ```

> ⚠️ **Security note:** the Gemini API key is currently hard-coded in the node URL.
> Consider moving it to a credential or environment variable.

---

### 4. Validate AI
**Type:** `n8n-nodes-base.code` (JavaScript)

- **Input:** The Gemini API response from **AI Scoring**, plus a back-reference to the
  original cleaned lead via `$('Clean Data').first().json`.
- **Processing:** Safely extracts and `JSON.parse`s the AI text. If extraction or
  parsing fails, it falls back to a default verdict (`score: 5`, `action: nurture`)
  so the workflow never breaks — this is the "resilient" part.
- **Output:** The cleaned lead merged with the AI verdict:
  ```json
  {
    "name": "Jane Doe", "email": "jane@acme.com", "domain": "acme.com",
    "company": "Acme", "message": "...", "source": "website",
    "ai_score": 8,
    "ai_reasoning": "Strong SaaS fit",
    "action": "hot"
  }
  ```

---

### 5. Route Leads
**Type:** `n8n-nodes-base.switch`

- **Input:** The validated lead object; branches on the `action` field.
- **Processing / Output branches:**
  - **Output 0 — `hot`:** `action === "hot"`
  - **Output 1 — `nurture`:** `action === "nurture"`
  - **Output 2 — `disqualify`:** `action === "disqualify"`
- The same item passes through unchanged on the matching branch.

> Note: Only the **hot** branch (output 0) is currently wired to a downstream node.
> The `nurture` and `disqualify` branches have no connected nodes yet.

---

### 6. Append row in sheet
**Type:** `n8n-nodes-base.googleSheets` (append)

- **Input:** Items from the **hot** branch of **Route Leads**.
- **Processing:** Appends a row to the **HOT** sheet of the "Lead Router" spreadsheet
  using auto-mapped input data. Mapped columns: `name`, `email`, `company`,
  `ai_score`, `ai_reasoning`.
- **Output:** The Google Sheets API response confirming the appended row(s).

---

## Sample webhook payload & response

### Incoming request (what you POST to the Webhook)

```http
POST /webhook/6577c89e-6495-4029-8b49-60cc509c5434 HTTP/1.1
Content-Type: application/json
```

```json
{
  "user_info": {
    "full_name": "jane DOE",
    "email_address": "JANE@ACME.COM"
  },
  "form_data": {
    "company_name": "Acme Inc",
    "message_text": "We're evaluating tools for our sales team and would like a demo."
  },
  "metadata": {
    "source": "website-contact-form"
  }
}
```

### After Clean Data

```json
{
  "name": "Jane Doe",
  "email": "jane@acme.com",
  "domain": "acme.com",
  "company": "Acme Inc",
  "message": "We're evaluating tools for our sales team and would like a demo.",
  "source": "website-contact-form"
}
```

### Gemini verdict text (parsed inside Validate AI)

```json
{
  "score": 9,
  "reasoning": "Clear buying intent, business email domain, requests a demo — strong B2B SaaS fit.",
  "action": "hot"
}
```

### After Validate AI (routed by Route Leads)

```json
{
  "name": "Jane Doe",
  "email": "jane@acme.com",
  "domain": "acme.com",
  "company": "Acme Inc",
  "message": "We're evaluating tools for our sales team and would like a demo.",
  "source": "website-contact-form",
  "ai_score": 9,
  "ai_reasoning": "Clear buying intent, business email domain, requests a demo — strong B2B SaaS fit.",
  "action": "hot"
}
```

### Row appended to the HOT sheet

| name     | email          | company  | ai_score | ai_reasoning                                             |
|----------|----------------|----------|----------|---------------------------------------------------------|
| Jane Doe | jane@acme.com  | Acme Inc | 9        | Clear buying intent, business email domain, requests... |

> The Webhook node responds to the caller with the default n8n acknowledgement
> (HTTP `200`) unless you configure a custom "Respond to Webhook" behavior.

---

## Setup notes

- **Google Sheets credential** is required for the append node.
- **Gemini API key** should be secured (see security note above).
- The workflow is currently **unpublished** — publish/activate it to make the webhook live.