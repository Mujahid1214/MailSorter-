# MailSorter: AI Email Triage with n8n and OpenAI

An n8n workflow that reads incoming customer emails, classifies them, scores urgency and sentiment, writes a draft reply, logs everything to Google Sheets, and alerts a manager when a customer is very angry.

> **Human in the loop:** MailSorter never sends a reply on its own. Draft replies are saved in Gmail Drafts so a person always reviews them first.

<!-- Add a screenshot of your n8n workflow here -->
<!-- ![Workflow screenshot](docs/workflow.png) -->

---

## The Problem

Businesses that receive customer emails face the same issues every day:

- Someone has to read and sort every email by hand.
- Angry customers wait too long because their email is buried under dozens of others.
- Replies are written from scratch again and again.
- Nobody can easily see patterns, such as many complaints about late delivery.

## The Solution

MailSorter automates the first-response work:

1. A new email arrives in Gmail.
2. OpenAI reads it and returns structured JSON: category, urgency, sentiment, summary, and a draft reply.
3. The result is validated, so broken AI output never crashes the workflow.
4. Every email is logged to a Google Sheet.
5. Urgent emails (urgency 4 or 5) trigger an alert email to the manager.
6. A draft reply is saved to Gmail Drafts (spam is skipped).

## Features

- **Classification** into `complaint`, `refund`, `question`, `spam`, or `other`
- **Urgency score** from 1 (low) to 5 (critical)
- **Sentiment detection**: `calm`, `upset`, or `very_angry`
- **One-line summary** of each email
- **Draft replies** that never invent policies, prices, or promises
- **Manager alerts** for urgent emails
- **Google Sheets log** of every processed email
- **Safe failure handling**: invalid AI output is marked `needs_review`, and failed API calls are retried once

## Architecture

```
Gmail Trigger
     |
     v
Prepare Email (build the OpenAI request)
     |
     v
OpenAI Classify (gpt-4o-mini, JSON mode, retry on fail)
     |
     v
Parse and Validate (check JSON, set defaults, flag bad output)
     |
     +--> Log to Google Sheet
     |
     +--> Urgent? (urgency >= 4) --> Alert Manager (Gmail)
     |
     +--> Needs Draft? (not spam) --> Save Draft Reply (Gmail)
```

## Tech Stack

| Part | Tool |
|---|---|
| Automation | n8n (self-hosted community edition) |
| AI model | OpenAI `gpt-4o-mini` |
| Email | Gmail (trigger, alerts, drafts) |
| Logging | Google Sheets |

The only paid service is the OpenAI API, and its cost is very low: a few cents for hundreds of emails.

## Getting Started

### Prerequisites

- An n8n instance (self-hosted or cloud)
- A Gmail account
- A Google account for Google Sheets
- An OpenAI API key with some credit

### Setup

1. **Import the workflow.** In n8n, create a new workflow, then choose **Import from file** and select `MailSorter-n8n-workflow.json`.

2. **Create the Google Sheet.** Add these headers in row 1, exactly as written:

   `Date` | `Sender` | `Subject` | `Category` | `Urgency` | `Sentiment` | `Summary` | `Status`

3. **Set the Sheet ID.** In the **Log to Google Sheet** node, replace `YOUR_GOOGLE_SHEET_ID` with the ID from your sheet's URL (the text between `/d/` and `/edit`).

4. **Add credentials.** Open each node that shows a red warning and select or create credentials:
   - Gmail OAuth2: **Gmail Trigger**, **Alert Manager**, **Save Draft Reply**
   - Google Sheets OAuth2: **Log to Google Sheet**
   - OpenAI API: **OpenAI Classify**

5. **Set the manager email.** In **Alert Manager**, replace `manager@example.com` with a real address.

6. **Activate the workflow** and send a few test emails to the connected Gmail inbox.

> Never commit API keys or credentials to this repository. n8n stores them securely in its own credential manager.

## Testing

Send these kinds of emails from another account:

| Test email | Expected result |
|---|---|
| Angry complaint about a late order | `complaint`, urgency 4-5, `very_angry`, manager alert sent |
| Refund request for a damaged product | `refund`, urgency about 3, draft saved |
| Simple question about delivery | `question`, urgency 1-2, `calm`, draft saved |
| Lottery or prize scam | `spam`, no draft saved |
| Vague message with no details | `other` or `question`, draft asks a clarifying question |

## Results

Tested on a set of **[30]** emails (**[10]** complaints, **[8]** refunds, **[7]** questions, **[5]** spam), labeled by hand.

| Metric | Result |
|---|---|
| Classification accuracy | **[X]%** ([N] of 30 correct) |
| Urgent emails correctly flagged | **[N] of [M]** |
| Average triage time per email (manual) | **[A]** |
| Average triage time per email (MailSorter) | **[B]** |
| Emails marked `needs_review` | **[N]** |
| OpenAI cost for the full test | **$[cost]** |

## Limitations

- Accuracy depends on the prompt and the quality of the email text. Very short or unclear emails are harder to classify.
- Draft replies are general. For policy-specific answers, add your company's rules to the prompt.
- The workflow polls Gmail every minute, so there is a delay of up to about one minute.
- Only the first 4,000 characters of each email are sent to the model.

## Future Improvements

- Give the model a knowledge document (refund policy, FAQ) so drafts follow real company rules
- Weekly summary email with email counts, urgent cases, and top complaint topics
- Support for attachments and non-English emails
- Dashboard for categories, response times, and trends

## Author

**Mujahid Hussain**, Python developer and automation specialist

- GitHub: [Mujahid1214](https://github.com/Mujahid1214)
- LinkedIn: [mujahid-hussain-b20518226](https://www.linkedin.com/in/mujahid-hussain-b20518226)

## License

MIT License. Feel free to use and adapt this workflow.
