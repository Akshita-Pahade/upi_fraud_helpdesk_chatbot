# UPI Fraud Helpdesk Chatbot (PayGuard Assistant)

A RAG chatbot that answers UPI fraud, refund and complaint questions from official RBI and NPCI documents. It names the source in every answer and says so when something is not in its documents.

Built with **n8n**, **Google Gemini**, **Pinecone**, **Google Drive** and **Google Sheets**. No code was written; the whole agent is two n8n workflows.

**Demo video:** [add your YouTube (unlisted) link here]

![Chatbot workflow](screenshots/workflow-chatbot.png)

---

## The problem

After a UPI fraud, people do not know three things: what they must do first, how much money they can lose, and where to complain. The answers exist in RBI and NPCI documents, but those documents are long and hard to read. This chatbot reads them and answers in plain English.

## What it does

- Answers questions from the RBI and NPCI documents only.
- Names the source in each answer, for example *(RBI circular, Para 9)*.
- Remembers the conversation, so follow-ups like "and if it takes 6 days?" work.
- Says "This is not in my documents" when it does not know, instead of guessing.
- Labels anything from outside the documents as *General guidance (not from my documents)*.
- Never promises a refund and never asks for a PIN or OTP.
- Refuses off-topic questions and harmful requests.
- Logs every chat to a Google Sheet.

## How it works

The project has two n8n workflows.

**1. Push Data to Vector Database** (`workflows/1-push-data-to-vector-database.json`)

```
Google Drive Trigger -> Download file -> Pinecone Vector Store
                                          |- Embeddings Google Gemini
                                          |- Default Data Loader
```

When a PDF is added to a Google Drive folder, n8n downloads it, splits it into chunks, turns the chunks into embeddings with Gemini, and stores them in Pinecone.

**2. UPI Chatbot** (`workflows/2-upi-chatbot.json`)

```
When chat message received -> AI Agent -> Append row in Google Sheet -> Edit Fields
                               |- Google Gemini Chat Model
                               |- Simple Memory
                               |- Tool: search_knowledge_base (Pinecone Vector Store)
                                     |- Embeddings Google Gemini
```

The AI Agent searches Pinecone first, then answers from what it finds. The system prompt is in `docs/system-prompt.txt`.

## Documents used

| Document | Source |
|---|---|
| RBI circular RBI/2017-18/15, 6 July 2017: *Customer Protection – Limiting Liability of Customers in Unauthorised Electronic Banking Transactions* | [RBI website](https://www.rbi.org.in/commonman/english/scripts/Notification.aspx?Id=2336) |
| NPCI *Dispute Redressal Mechanism* (UPI) | [add the npci.org.in page link you used] |

The PDFs are not stored in this repo. Download them from the official pages above.

## Tech stack

| Part | Tool |
|---|---|
| Workflow builder | n8n |
| Chat model and embeddings | Google Gemini API (free tier) |
| Vector database | Pinecone |
| Document storage | Google Drive |
| Chat log | Google Sheets |

## How to run it

1. Install n8n (`npx n8n`) or use n8n Cloud.
2. Import both files from the `workflows/` folder (menu, then **Import from file**).
3. Add your own credentials: Google Drive, Google Sheets, Gemini API key and Pinecone API key. Credentials are not included in the exported files.
4. Create a Pinecone index. Its dimension must match your Gemini embedding model, and both workflows must use the same index.
5. Put the two documents in a Google Drive folder, then point the Drive Trigger in workflow 1 at that folder.
6. Run workflow 1 once to load the documents.
7. Open workflow 2 and click **Open chat**.

## Test results

I tested the bot with questions whose answers I know from the documents, plus questions it should refuse.

| Question | Source in document | Result |
|---|---|---|
| I see an unauthorised transaction. What should I do first? | RBI Para 5 | Correct |
| I report within 3 working days. What is my liability? | RBI Table 2 | Correct |
| And if it takes 6 days? (follow-up) | RBI Table 1 and 2 | Correct |
| How fast must the bank return my money? | RBI Para 9 and 10 | Correct |
| Where do I first raise a UPI complaint? | NPCI mechanism | [add result] |
| [add more rows] | | |

**Score:** [X] of [Y] questions correct on questions that are in the documents. [Z] of [W] behaved correctly on questions that should be refused or marked "not in my documents".

## Limitations

- It uses only two documents, so it cannot answer questions about scam types, prevention tips or UPI basics.
- The RBI circular is from 2017. Rules may have changed, so users should confirm with their bank.
- The circular applies to scheduled commercial banks, small finance banks and payments banks, not to every institution.
- The Gemini free tier has daily request limits, so heavy testing can hit a rate-limit error.
- Advice such as calling 1930 is not in the documents. The bot labels it as general guidance.
- It gives information, not legal or financial advice. It cannot access bank accounts, trace payments or file complaints.

## Future scope

- Telegram or WhatsApp version, so people can use it on their phones.
- Hindi and Marathi support.
- More official documents: RBI awareness booklet, Ombudsman scheme, cyber crime reporting pages.
- A step-by-step "report my fraud" flow.
- A feedback button, so users can mark wrong answers.

## Disclaimer

This is a student project. It is not affiliated with RBI, NPCI or any bank. Do not treat its answers as legal or financial advice.

---

Built by Akshita Pahade as an AI mini project.
