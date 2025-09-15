<img width="411" height="294" alt="image" src="https://github.com/user-attachments/assets/79a5d226-7a6f-4b68-941c-39e585a5c6c1" />
<img width="424" height="210" alt="image" src="https://github.com/user-attachments/assets/d9551545-d5bb-459c-90b1-2c4fe7dcb976" />

This repository contains two n8n workflow JSONs: one to ingest an IT Customer Support Handbook from Google Drive into a Pinecone vector store, and another to power a Telegram-based IT support assistant constrained to internal policies and procedures defined by the handbook prompt boundaries.[1][2]

### Repository contents
- IT-Support-Handbook.json: Ingests a handbook PDF from Google Drive, chunks it, embeds with OpenAI (512-dim), and upserts vectors to a Pinecone index named studentshandbooks.[1]
- Telegram-IT_Support.json: Telegram bot workflow using an n8n LangChain Agent with OpenAI chat model, memory buffer, and message delivery back to Telegram users, with a strict system prompt scope and escalation flow.[2]

### Architecture overview
- Data pipeline: Google Drive Trigger → Download file → Recursive Character Text Splitter → Default Data Loader → OpenAI Embeddings → Pinecone Vector Store.[1]
- Bot pipeline: Telegram Trigger → AI Agent (OpenAI Chat + Memory) → Telegram Send Message to user chat id.[2]

### Key features
- Automated file watch and re-index: Google Drive Trigger monitors a specific fileId and reprocesses on updates for near-real-time vector refresh.[1]
- Chunking optimized for retrieval: Recursive character splitting with chunkSize=500 and chunkOverlap=50 to balance context and recall.[1]
- Vector storage: OpenAI embeddings with dimensions=512 inserted into Pinecone index studentshandbooks in insert mode.[1]
- Guardrailed bot: System prompt enforces supported IT support topics, out-of-scope refusal, and an end-of-conversation “Was your issue resolved?” check, with escalation guidance.[2]

### Security and compliance
- Credentials are referenced by name in n8n (Google Drive OAuth2, OpenAI, Pinecone, Telegram) and must be configured in the target instance; do not commit secrets to the repository.[2][1]
- Telegram and Drive triggers rely on webhook/endpoints; restrict access via IP allowlists/VPN and protect tokens; rotate credentials regularly per security policy.[2][1]
- Ensure OpenAI/Pinecone usage complies with organizational data handling and retention requirements before indexing internal documents.[2][1]

### Prerequisites
- n8n v1.x with LangChain community nodes available and enabled in the instance.[1][2]
- Accounts and credentials: Google Drive OAuth2, OpenAI API, Pinecone API, Telegram Bot token configured inside n8n Credentials.[2][1]
- Pinecone index created: studentshandbooks with dimension 512 and an appropriate similarity metric (cosine or dot-product) to match embeddings.[1]
- Access to Google Drive file: IT_Customer_Support_Handbook.pdf with fileId 1WhN4gIUiXZhdWf6G9mP0JiPRhEvjMbpU, or update the workflow parameter to a different file.[1]

### Installation
- Import both JSONs via n8n: Settings → Workflows → Import from file for IT-Support-Handbook.json and Telegram-IT_Support.json.[2][1]
- Create and map credentials in n8n: Google Drive OAuth2, OpenAI, Pinecone, Telegram; ensure node credential selectors point to the correct accounts.[2][1]
- Adjust node parameters as needed: fileId, Pinecone index name, embedding dimensions, and Telegram webhook configuration.[1][2]
- Test in manual mode, verify successful executions, then activate the workflows for continuous operation.[2][1]

### Configuration details
- IT-Support-Handbook.json.[1]
  - Google Drive Trigger: triggerOn=specificFile, polls everyMinute for fileId 1WhN4gIUiXZhdWf6G9mP0JiPRhEvjMbpU. 
  - Download file: fetches latest IT_Customer_Support_Handbook.pdf from Google Drive. 
  - Recursive Character Text Splitter: chunkSize=500, chunkOverlap=50. 
  - Embeddings OpenAI: dimensions=512 configured in the embeddings node. 
  - Default Data Loader: consumes binary data for downstream vectorization. 
  - Pinecone Vector Store: index=studentshandbooks, mode=insert to upsert embeddings and documents.
- Telegram-IT_Support.json.[2]
  - Telegram Trigger: listens for message updates. 
  - AI Agent: receives message text, uses a systemMessage limiting supported topics and enforcing an escalation path; has output parser enabled. 
  - OpenAI Chat Model: model set to gpt-3.5-turbo via the n8n OpenAI Chat node. 
  - Simple Memory: buffer window keyed by message sessionKey for short contextual continuity. 
  - Send a text message: posts the AI Agent’s output to chat.id from the incoming Telegram message.

### System prompt policy (bot)
- Supported topics: purpose/scope, support team responsibilities, supported tools/channels (ConnectWise, Outlook, MS Teams), SLAs and prioritization, ticket lifecycle (New → Closed), troubleshooting tiers (Tier 1–3), communication best practices, escalation protocols, KB contribution guidelines, IT security/compliance essentials, operational do’s/don’ts, and continuous improvement.[2]
- Out-of-scope handling: respond with “I’m here to help you with matters specifically related to our IT Customer Support policies and procedures. Please rephrase your question to align with these topics.”.[2]
- End-of-conversation: always ask “Was your issue resolved with the above information?” and branch responses; if “No”, direct to submit details via a Google Form and email the response to pulin2411@gmail.com per the defined process.[2]

### Requirements
- Runtime: Deployed n8n instance with network access to Google APIs, Telegram, OpenAI, and Pinecone endpoints; stable HTTPS public webhook URL for Telegram.[1][2]
- Access: Service account or OAuth scopes permitting Drive file watch and download; Telegram bot token configured; OpenAI and Pinecone API keys with appropriate quotas.[1][2]
- Data: Source handbook PDF present and maintained in Google Drive; Pinecone index provisioned with 512-dimension configuration prior to ingestion.[1]
- Policy: Organizational approval for using LLMs and vector databases with internal documents; compliance review for prompt rules and escalation handling.[1][2]

### Running locally (Docker example)
- Launch n8n via Docker with mounted volume for persistence; expose N8N_PORT and set WEBHOOK_URL to a public URL (e.g., via reverse proxy).[2][1]
- Set environment variables: N8N_ENCRYPTION_KEY, N8N_HOST, N8N_PORT, WEBHOOK_URL, and enable community nodes for LangChain integrations if required by the build.[1][2]
- Import both workflows via the n8n UI and link credentials, then activate.[2][1]

### Operations
- Updating the handbook: replace or update the Google Drive file; the trigger detects changes and reprocesses to refresh vectors.[1]
- Monitoring: review n8n execution logs; configure error workflows and notifications for failures in Drive download, embeddings, or Pinecone upserts.[2][1]
- Rate limits: consider OpenAI token usage, Pinecone QPS, and Telegram message throttling; add retry/backoff as needed.[1][2]

### Troubleshooting
- Pinecone dimension mismatch: ensure embeddings dimension equals index dimension (512) to prevent upsert errors.[1]
- Google Drive permission errors: validate OAuth scopes and that the n8n Google Drive credential has access to the fileId.[1]
- Telegram no response: verify bot token, webhook reachability, and that the Telegram Trigger is active; check chat.id mapping in the Send Message node.[2]
- Large PDFs or poor retrieval: tune chunkSize/chunkOverlap; consider adding content-type-specific loaders if future documents are not PDFs.[1]

### Extensibility
- Retrieval grounding: add a RetrievalQA or ConversationalRetrievalQA chain to the bot to answer strictly from Pinecone contents instead of policy-only prompts.[2][1]
- Alternative models: swap OpenAI embeddings/chat with other providers supported by n8n LangChain nodes; ensure index dimensions match.[2][1]
- Analytics: log user questions and responses to a database or data warehouse for QA review and continuous improvement.[2]

### About Author
Pulin Shah — Lead IT Support Coordinator | IT Service Delivery | n8n Automations |ML|DL|Data Science|Prompt Enigneering|RAG|Gen-AI|EDA

IT service delivery leader with 20+ years across incident, change, and problem management, PSA/RMM operations (ConnectWise), and endpoint security. Designs and operates Service Desk workflows with SLA rigor, and builds support automations using n8n, LLMs, and vector search for policy-aligned responses.

Leads Service Desk operations: ticket triage, scheduling, vendor coordination, SLA governance, and reporting.

Builds LLM-powered assistants on Telegram with strict topic guardrails and end-of-conversation flows.

Implements RAG pipelines: Google Drive ingestion, OpenAI embeddings (512-dim), Pinecone indexing, and chunking strategies for retrieval quality.

Experienced with ESET/SentinelOne endpoints, Microsoft stack, AWS (Solutions Architect), and ITIL-based practices for change/incident/request management.

Links

GitHub: https://github.com/Pulin2411

LinkedIn: linkedin.com/in/pulin-shah-741212b

Email: pulin2411@gmail.com



