# Podcast 29 — AI/ML Services

**Length target**: 15 min
**Repo references**: `domain-2-new-solutions/35-bedrock.md`, `36-sagemaker.md`, `48-ml-services.md`

## Topic & Scope

Bedrock (generative AI / foundation models), SageMaker (custom ML platform), plus the pre-trained AI service bundle (Comprehend, Polly, Transcribe, Rekognition, Textract, Translate, Lex, Kendra, Personalize, Fraud Detector). SAP-C02 mostly tests "which service for which input/output".

## Service Coverage Depth

**Deep**:
- Bedrock — foundation models, Knowledge Bases (RAG), Agents, Guardrails, Provisioned Throughput
- SageMaker — training, inference (real-time / async / batch / serverless), endpoints, multi-model + multi-container

**Brief — "input/output recognition"**:
- Comprehend (text NLP)
- Polly (text-to-speech)
- Transcribe (speech-to-text)
- Rekognition (image/video)
- Textract (OCR + forms)
- Translate (translation)
- Lex (chatbots)
- Kendra (enterprise search)
- Personalize (recommendations)
- Fraud Detector

## Structured Outline

1. **Open (30s)** — "Generative AI is on the exam. Plus SageMaker and the pre-trained AI bundle. Memorize: which input becomes which output."
2. **Bedrock (5 min)** — foundation models from Anthropic / Meta / Mistral / AI21 / Cohere / Amazon Titan / Stability, Knowledge Bases (RAG with vector DB), Agents (tool use), Guardrails (content filtering), Provisioned Throughput, Model evaluation
3. **SageMaker (4 min)** — Studio (IDE), training (managed compute), inference modes (real-time, async, batch, serverless), endpoint deployment (single model, multi-model, multi-container, inference pipeline), Feature Store, Model Monitor, Clarify (bias detection), Pipelines (ML CI/CD), Ground Truth (labeling), JumpStart (model zoo)
4. **Pre-trained AI services Quick Round (4 min)** — verbalize each as "input → output": Comprehend (text→sentiment/entities), Polly (text→speech), Transcribe (speech→text), Rekognition (image/video→labels), Textract (document→structured fields), Translate (text→translated text), Lex (utterance→bot response), Kendra (query→ranked documents), Personalize (interactions→recommendations), Fraud Detector (transaction→risk score)
5. **Decision Matrix (1 min)**
6. **Rapid-Fire Trap Drill (60s)**

## Must-Mention Exam Tips

- Bedrock: managed access to foundation models via API; no infrastructure
- Bedrock Knowledge Bases: managed RAG with vector embeddings (Aurora pgvector / OpenSearch / Pinecone)
- Bedrock Agents: tools + reasoning loop; calls Lambda or APIs
- Bedrock Guardrails: content policy + PII redaction + topic blocking + grounding
- Provisioned Throughput: dedicated capacity for production at lower per-token cost
- SageMaker training: built-in algorithms, BYO container, Spot training for cost
- SageMaker inference: real-time (low latency, always on), async (queue, large payloads, GPU), batch (S3 in/out), serverless (cold-start tolerant)
- SageMaker Multi-Model Endpoint: many models on one endpoint, lazy-load
- SageMaker Feature Store: online + offline feature serving
- SageMaker Pipelines: ML CI/CD with model registry + approval workflow
- SageMaker Clarify: bias detection + explainability
- SageMaker Model Monitor: data drift detection
- SageMaker Ground Truth: managed labeling workflow
- SageMaker JumpStart: pre-trained models, one-click deploy
- Comprehend: sentiment, entities, key phrases, classification (custom too)
- Comprehend Medical: HIPAA medical text
- Polly: TTS, 60+ voices, SSML, neural voices
- Transcribe: ASR, real-time + batch, speaker ID, custom vocab, PII redaction
- Rekognition: image/video labels, faces, content moderation, custom labels
- Textract: OCR + forms + tables, document Q&A queries
- Translate: 75+ languages, custom terminology, document translation preserves format
- Lex: chatbots — same engine as Alexa
- Kendra: enterprise search with NLP
- Personalize: recommendation engine
- Fraud Detector: ML-based fraud risk scoring

## Must-Mention Exam Traps

- EXAM TRAP: Bedrock is for FOUNDATION MODELS — not for training your own from scratch (use SageMaker)
- EXAM TRAP: Bedrock data is NOT used to train base models — your prompts/responses stay private
- EXAM TRAP: SageMaker endpoints have cold starts on serverless mode — for real-time low-latency use real-time endpoint
- EXAM TRAP: SageMaker async inference for large payloads + variable latency — uses queue + SNS notification
- EXAM TRAP: Comprehend ≠ Textract — Comprehend analyzes text; Textract extracts structured data from documents
- EXAM TRAP: Lex builds bots; for free-form generation use Bedrock
- EXAM TRAP: Kendra is search — for RAG pair with Bedrock
- EXAM TRAP: Personalize needs interaction history — won't work cold-start
- EXAM TRAP: Rekognition Custom Labels ≠ SageMaker — Custom Labels is no-code; SageMaker is full ML platform
- EXAM TRAP: SageMaker Studio Notebooks are billed when running — stop when not in use
- EXAM TRAP: Bedrock Knowledge Base uses vector DB — OpenSearch Serverless is the default cheap option
- EXAM TRAP: SageMaker multi-model endpoint requires models in S3 + container that supports multi-model
- EXAM TRAP: Some AI services are not in every Region — check before designing

## Key Decision Matrix

| Input → Output | Pick |
|---|---|
| Text → sentiment / entities | Comprehend |
| Text → speech | Polly |
| Speech → text transcript | Transcribe |
| Image / video → objects / labels | Rekognition |
| Document → structured form data | Textract |
| Text → translated text | Translate |
| User → chatbot reply | Lex |
| Query → ranked enterprise docs | Kendra |
| User events → recommendation | Personalize |
| Transaction → fraud score | Fraud Detector |
| Prompt → generated text / code / image | Bedrock |
| Custom ML model training | SageMaker |
| Generative AI with grounded enterprise docs | Bedrock + Knowledge Base |
| LLM agent with tool calls | Bedrock Agents |
| Block toxic / off-topic LLM output | Bedrock Guardrails |

## Tone & Style

- For each pre-trained AI service, give the "input → output" sentence and one use case
- Bedrock + SageMaker get the deep treatment; everything else is recognition
- Repeat "Bedrock for foundation models, SageMaker for your own"

## Rapid-Fire Closer

"OCR + forms?" — "Textract." "Text → speech?" — "Polly." "Speech → text?" — "Transcribe." "Chatbot?" — "Lex." "Enterprise search?" — "Kendra." "Recommendations?" — "Personalize." "Foundation models?" — "Bedrock." "Custom ML?" — "SageMaker."
