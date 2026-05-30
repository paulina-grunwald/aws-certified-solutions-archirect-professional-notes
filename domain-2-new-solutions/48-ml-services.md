# AWS AI/ML Services (Consolidated Reference)

> **One-page reference for AWS's pre-trained AI/ML services that SAP-C02 tests as "right tool for the job" distractors. These are managed APIs — no ML expertise required. Pick the service that matches the input type + output: text understanding (Comprehend), text to speech (Polly), speech to text (Transcribe), image/video analysis (Rekognition), document text/forms (Textract), translation (Translate), conversational bots (Lex), enterprise search (Kendra), recommendations (Personalize), fraud detection (Fraud Detector). Bedrock (foundation models / generative AI) and SageMaker (custom ML) are separate notes.**

Maps to: **Domain 2.5 — High-performance**, **Domain 4.3 — Modernization**, **Domain 4.4 — Modernization opportunities**

---

## Amazon Comprehend (Natural Language Processing)

- **Text analysis** API: entities, key phrases, sentiment, language detection, syntax, topic modeling, PII detection
- **Comprehend Medical** = HIPAA-eligible variant for medical text (ICD-10 codes, medications, conditions)
- **Custom classification + entity recognition** — train your own on labeled data
- Use cases: tag support tickets, sentiment analysis, PII redaction, document classification

**Pick when**: "Analyze sentiment / detect entities / classify text" → **Comprehend**.

---

## Amazon Polly (Text-to-Speech)

- **TTS** in 60+ voices, 30+ languages
- **Neural voices** (high quality) + standard voices
- **SSML markup** for prosody control (pauses, emphasis)
- **Speech marks** — word-level timing (useful for karaoke / subtitle sync)
- Real-time + batch synthesis
- Output: MP3, OGG, PCM

**Pick when**: "Convert text to spoken audio" → **Polly**.

---

## Amazon Transcribe (Speech-to-Text)

- **ASR**: speech to text
- Real-time (streaming) + batch
- **Speaker identification** (diarization)
- **Custom vocabulary** for industry jargon
- **Transcribe Medical** = HIPAA medical variant
- **Transcribe Call Analytics** — sentiment + summarization for call center recordings
- Languages: 100+
- **PII redaction** built-in

**Pick when**: "Convert audio recording to text transcript" → **Transcribe**. "Real-time captioning" → **Transcribe streaming**.

---

## Amazon Rekognition (Image + Video Analysis)

- **Image** API: object/scene detection, faces (detect / compare / search), text in images, content moderation, celebrity recognition, PPE detection
- **Video** API: same + video-level activity (face tracking, person path, label timestamps)
- **Custom Labels**: train on your own images
- **Face liveness** (anti-spoofing for face login)

**Pick when**: "Detect objects / faces / inappropriate content in images or video" → **Rekognition**.

---

## Amazon Textract (Document OCR + Forms)

- **OCR + structure extraction** for scanned documents, PDFs
- Beyond OCR: extracts **forms (key-value pairs)** and **tables** with structure preserved
- **Queries**: ask "what is the total amount?" against the document
- **Signatures detection**
- **AnalyzeID / AnalyzeExpense / AnalyzeLending** specialized pipelines
- Async for multi-page documents

**Pick when**: "Extract data from scanned forms / invoices / IDs / tables" → **Textract**.

---

## Amazon Translate (Machine Translation)

- **75+ languages**
- Real-time + batch
- **Custom Terminology** + **Active Custom Translation** (parallel data fine-tuning)
- **Document translation** (preserves formatting)
- Output: same format as input

**Pick when**: "Translate text between languages" → **Translate**.

---

## Amazon Lex (Conversational Bots)

- **Chat + voice bots**: same engine as Alexa
- **Intents + slots + utterances** model
- Integrates with **Connect** for call center bots
- **Lambda fulfillment** for backend logic
- Multi-language

**Pick when**: "Build chatbot / voice IVR" → **Lex**.

---

## Amazon Kendra (Enterprise Search)

- **ML-powered search** over enterprise documents
- **Natural language queries**: "What's our PTO policy?"
- Connectors: S3, SharePoint, Salesforce, ServiceNow, Confluence, Google Drive, RDS, Slack, GitHub, etc.
- **Faceted search** + relevance tuning
- **Q&A** via passage extraction
- Common pairing with **Bedrock** for RAG (retrieval-augmented generation)

**Pick when**: "Search internal documents with natural language" → **Kendra**. "Build a RAG app with foundation model" → **Bedrock + Kendra**.

---

## Amazon Personalize (Recommendations)

- **Recommendation engine** as a service
- Same tech behind Amazon.com recommendations
- Input: user-item interactions (e.g., views, purchases, ratings)
- Output: personalized recommendations + similar items + user segments
- Real-time + batch
- Use cases: "people who bought X also bought Y", content recommendations, email personalization

**Pick when**: "Recommend products / content to users" → **Personalize**.

---

## Amazon Fraud Detector

- **ML-based fraud risk scoring**
- Pre-built models for: online fraud, transaction fraud, account takeover
- Custom models on your historical data
- Real-time scoring API
- Use cases: payment fraud, signup fraud, account takeover

**Pick when**: "Detect fraudulent transactions / accounts" → **Fraud Detector**.

---

## Decision Matrix

| Input | Output | Service |
|---|---|---|
| Text | Sentiment / entities / classification | **Comprehend** |
| Text | Speech (audio) | **Polly** |
| Speech (audio) | Text transcript | **Transcribe** |
| Image / video | Objects / faces / labels | **Rekognition** |
| Document (PDF / scan) | Structured data (forms, tables) | **Textract** |
| Text in language A | Text in language B | **Translate** |
| User input | Bot response | **Lex** |
| Documents + query | Search result | **Kendra** |
| User-item events | Personalized list | **Personalize** |
| Transaction / event | Fraud score | **Fraud Detector** |
| Prompt / instruction | Generated text / image | **Bedrock** (separate note) |
| Custom ML model | Inference | **SageMaker** (separate note) |

---

## Common Patterns

### Customer support automation
- **Lex** bot for first-touch resolution
- **Comprehend** to classify tickets
- **Translate** for multilingual support
- **Connect** integrates Lex into call center

### Document automation
- **Textract** extracts data from invoices / forms
- **Comprehend** classifies document type + redacts PII
- Output to S3 + DynamoDB

### Accessible content
- **Polly** narrates articles for visually-impaired
- **Transcribe** captions video content
- **Translate** localizes content
- **Rekognition** generates alt-text for images

### Generative AI with grounded knowledge
- **Bedrock** for LLM (Claude, Llama, etc.)
- **Kendra** for retrieval (RAG)
- **Comprehend** for response moderation

### Real-time recommendations
- **Personalize** for ML recommendations
- Events streamed via **Kinesis**
- Recommendations cached in **DynamoDB**
- Served via **API Gateway + Lambda**

### Fraud-aware checkout
- **Fraud Detector** scores each transaction
- Score > threshold → step-up auth
- Audit trail to **S3** + **CloudWatch Logs**

---

## Pricing (rough orders of magnitude)

- **Comprehend**: $0.0001 per 100 characters
- **Polly**: $4 per 1M characters (neural); $4 per 1M characters (standard)
- **Transcribe**: $0.024 per minute (first 250K minutes)
- **Rekognition**: $1 per 1,000 images analyzed
- **Textract**: $1.50 per 1,000 pages (forms + tables)
- **Translate**: $15 per 1M characters
- **Lex**: $0.004 per request (speech) / $0.00075 per request (text)
- **Kendra**: $810+/month for Developer Edition; Enterprise much higher
- **Personalize**: $0.0667 per training hour + recommendation requests
- **Fraud Detector**: $7.50 per 1,000 predictions

---

## Exam Tips

- "Sentiment / entities / topic modeling" → **Comprehend**
- "Text to speech" → **Polly**
- "Speech to text + captions" → **Transcribe**
- "Detect objects / faces in images" → **Rekognition**
- "OCR + structured forms / tables" → **Textract**
- "Translate text between languages" → **Translate**
- "Conversational bot" → **Lex**
- "Enterprise document search with NLP" → **Kendra**
- "Recommendation engine" → **Personalize**
- "Fraud risk scoring" → **Fraud Detector**
- Medical variants exist for **Comprehend** + **Transcribe** (HIPAA)

## Exam Traps

- **Comprehend ≠ Textract** — Comprehend analyzes text; Textract extracts text + structure from documents
- **Rekognition Custom Labels ≠ SageMaker** — Custom Labels is no-code; SageMaker is full ML platform
- **Lex builds bots, doesn't generate text** — for generative chat, use Bedrock
- **Kendra is search, not generation** — pair with Bedrock for RAG
- **Personalize needs interaction history** — won't work cold-start without seed data
- **Fraud Detector is a risk score, not auto-block** — you decide what to do with the score
- **All these services are Region-specific** — not in every Region; check before designing
- **Translate doesn't preserve PDF layout** — use Document Translation for that
- **Async vs sync APIs** — multi-page Textract / long Transcribe are async (job-based)
