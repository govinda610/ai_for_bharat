# System Design: SarvaSukh AI
**Architecture Pattern:** Hybrid Edge-Cloud Multi-Agent Health Platform  
**Design Philosophy:** Cloud-first for hackathon POC → Offline-capable for pilot → Full hybrid at scale.

---

## 1. High-Level Architecture

SarvaSukh AI operates as a **three-tier system** with progressive capability:

| Tier | Components | When Active |
|:---|:---|:---|
| **Edge Layer** (On-Device) | Entity extraction (Qwen 3), clinical rules (Rakshak), local storage (SQLite) | Pilot onwards (Phase 2+) |
| **Cloud Layer** (AWS) | ASR (Sarvam API), complex inference (Bedrock Claude 4.5), FHIR data store (HealthLake), sync | Always |
| **Intelligence Layer** | Time series forecasting (TimesFM), anomaly detection, dashboards (QuickSight) | Phase 3+ |

> **Hackathon POC:** Cloud-only architecture. On-device AI is a Phase 2 optimization.

---

## 2. Technology Stack

### 2.1 Complete Technology Mapping

| Layer | Component | Technology | Purpose |
|:---|:---|:---|:---|
| **Frontend** | Mobile App | React Native (Android-first) | Optimized for $80 phones, 2GB RAM |
| **Voice — ASR** | Speech-to-Text | **Sarvam AI Saaras V3** (cloud API) | 22 Indian languages, code-mixing, ~19% WER |
| **Voice — ASR Fallback** | Offline STT | **Whisper Tiny** (39M params, INT8 quantized) | ~100MB, lower accuracy but functional offline |
| **Voice — TTS** | Text-to-Speech | **Sarvam AI Bulbul V3** | 11 languages, 35+ natural voices |
| **Edge LLM** | On-Device Intelligence | **Qwen 3 0.5B** (4-bit GGUF via llama.cpp) | ~250MB, entity extraction. Latency: 5–10s on CPU |
| **Edge Fallback** | Alternate SLM | **Gemma 3** (quantized) | Hardware compatibility fallback |
| **Cloud LLM** | Complex Reasoning | **Amazon Bedrock (Claude 4.5)** | Complex entity extraction, summarization |
| **Time Series** | Forecasting | **TimesFM** (Google, open-source) | Zero-shot health forecasting |
| **Backend** | API Server | **FastAPI** on **AWS Lambda** | Serverless, auto-scaling |
| **API Gateway** | Gateway | **Amazon API Gateway** | Routing, rate limiting, auth |
| **DB (Edge)** | Local Storage | **SQLite** (WAL mode) | On-device patient data, offline queue |
| **DB (Cloud)** | Operational Data | **Amazon DynamoDB** | Patient records, visit logs, referrals |
| **Health Data** | FHIR Repository | **AWS HealthLake** (FHIR R4) | Standardized records, ABDM interop |
| **Storage** | Files & Media | **Amazon S3** | Audio logs, scanned documents, images |
| **Auth** | Identity | **Amazon Cognito** | Role-based access (ASHA/MO/Admin) |
| **Monitoring** | Observability | **Amazon CloudWatch** | Latency, errors, audit logs |
| **Dashboards** | Analytics | **Amazon QuickSight** | PHC/District health dashboards |
| **Messaging** | Async | **Amazon SNS / SQS** | Alert routing, background sync |

### 2.2 On-Device vs Cloud Decision Matrix

| Function | Hackathon POC | Pilot (Phase 2) | At Scale (Phase 3+) |
|:---|:---|:---|:---|
| **Speech-to-Text** | ☁️ Sarvam API only | ☁️ Sarvam API + 📱 Whisper Tiny (offline fallback) | Same |
| **Entity Extraction** | ☁️ Bedrock Claude 4.5 | 📱 Qwen 3 (5-10s) + ☁️ Bedrock for complex | 📱 Primary + ☁️ Complex |
| **Clinical Triage** | ☁️ Lambda (for demo) | 📱 Always on-device (sub-second) | 📱 Always on-device |
| **Data Storage** | ☁️ DynamoDB only | 📱 SQLite + ☁️ DynamoDB (sync) | Same |
| **Dashboards** | ☁️ QuickSight | ☁️ QuickSight | Same |
| **Forecasting** | ❌ Not in POC | ❌ Not in pilot | ☁️ TimesFM |

> **Key Clarification:** "Offline-capable" means the system **degrades gracefully** — core functions work on-device with reduced accuracy, and sync when connectivity returns. It does NOT mean "fully offline with identical performance." Sarvam Saaras V3 is a cloud API; offline ASR uses Whisper Tiny with lower dialect/code-mixing accuracy.

---

## 3. Multi-Agent Architecture

SarvaSukh AI uses **5 specialized agents**, each with clear input/output contracts and safety guardrails.

### Agent 1: "Sravana" (श्रवण — The Listener)

| Aspect | Detail |
|:---|:---|
| **Trigger** | ASHA presses "बोलिये" button or says wake word |
| **Task** | Audio capture → noise cancellation → transcription |
| **Online** | Sarvam Saaras V3 API (22 languages, code-mixing, ~19% WER) |
| **Offline Fallback** | Whisper Tiny (INT8, ~100MB). Supports Hindi, English. Lower accuracy for dialects/code-mixing. |
| **Output** | Transcribed text + confidence score + detected language |
| **Latency** | <3s (online), 5–8s (offline on 2GB device) |
| **Error Handling** | Confidence < 0.6 → *"Sunai nahi diya, phir se boliye"* (Didn't hear, please repeat) |

### Agent 2: "Lekhak" (लेखक — The Scribe)

| Aspect | Detail |
|:---|:---|
| **Task** | Structured entity extraction (NER) from transcribed text |
| **Online** | Amazon Bedrock Claude 4.5 (JSON mode, high accuracy) |
| **Offline** | Qwen 3 0.5B (4-bit GGUF via llama.cpp), JSON mode. Latency: 5–10s on CPU. |
| **Fallback** | Gemma 3 (quantized) for incompatible hardware |

**Example:**
```
Input:  "Sunita ka BP 150 by 90 hai, 7 mahine ki pregnant hai, chakkar aa rahe hain"
```
```json
{
  "patient_name": "Sunita",
  "vitals": { "bp_systolic": 150, "bp_diastolic": 90 },
  "symptoms": ["Dizziness"],
  "pregnancy": { "is_pregnant": true, "gestational_age_months": 7 },
  "visit_type": "ANC",
  "confidence": 0.97
}
```

| Error Handling | Action |
|:---|:---|
| Confidence < 0.8 on a field | Highlight field yellow, ask ASHA to confirm verbally |
| Missing critical field (e.g., BP value) | Prompt: *"BP kitna tha?"* |
| Extraction fails entirely | Fall back to manual touch-based form entry |

### Agent 3: "Rakshak" (रक्षक — The Guardian)

| Aspect | Detail |
|:---|:---|
| **Task** | Clinical risk flagging via **deterministic Python rules** (NOT AI prediction) |
| **Runs** | Always on-device (CPU, sub-second latency, zero cloud dependency) |
| **Safety** | Never diagnoses. Never prescribes. Only recommends referrals. |

**MVP Rule Set (Hackathon — 5 Rules):**

| Priority | Condition | Rule | Action |
|:---|:---|:---|:---|
| 🔴 CRITICAL | Pre-eclampsia | `BP_Sys>140 AND Pregnant` | Immediate PHC referral |
| 🔴 CRITICAL | Severe Low Birth Weight | `Child_Weight<2.0kg` | District Hospital referral |
| 🔴 CRITICAL | Severe Anemia | `Hb<7 g/dL` | Urgent referral + IV iron |
| 🟡 REPORT | Suspected Measles | `Temp>103°F AND Rash` | Auto-generate IDSP S-form |
| 🟢 REMINDER | Missed Vaccination | `Age>Due AND Vaccine_Not_Given` | Schedule reminder |

**Expanded Rule Set (Pilot — 15 Rules):** Adds Gestational Diabetes, Suspected TB, SAM, PPH risk, Neonatal danger signs, NCD screening thresholds.

**Rule Versioning:**
- Rules stored as **JSON config file** with version stamp.
- Fetched from cloud **weekly**, cached locally.
- No app update needed for rule changes.
- Audit log of all rule version changes.

**Priority System:** When multiple rules trigger → display in priority order: CRITICAL > HIGH > MODERATE > LOW.

### Agent 4: "Doot" (दूत — The Messenger / Sync Agent)

| Aspect | Detail |
|:---|:---|
| **Task** | Reliable data sync between device and cloud |
| **Online (Good)** | Immediate push → DynamoDB + HealthLake |
| **Online (Poor)** | Compress + batch → push with retry |
| **Offline** | Queue in SQLite → exponential backoff (30s → 1m → 5m → 15m → 1hr) |
| **After 5 failures** | Surface "Sync Failed" notification to ASHA. Data safe locally. |

**Conflict Resolution:**

| Scenario | Resolution |
|:---|:---|
| Same patient, different ASHAs, same time | Timestamp + device ID → both records kept, flagged in PHC "Review Needed" queue |
| ASHA edits offline + Doctor edits online | **Doctor's version always wins** for clinical fields (vitals, diagnoses) |
| Non-critical fields | Last-write-wins (timestamp-based) |
| SLA for conflict review | PHC MO reviews queue within 24 hours |

**Data Formats:**
- Audio: Opus codec (lossy, small size)
- Images: WebP compressed before S3 upload
- Records: JSON → FHIR R4 Bundle for ABDM

### Agent 5: "Darshak" (दर्शक — Observer / Vision Agent)

**Phase 2+** — Dual role: Vision/Multimodal + Population Analytics.

| Capability | Technology | Phase |
|:---|:---|:---|
| **OCR (Prescriptions)** | Qwen 3 VL (on-device) | Phase 2 |
| **Document Layout Detection** | Sarvam AI document understanding | Phase 2 |
| **Medical Image Pre-Analysis** | Claude 4.5 Vision (Bedrock, cloud) | Phase 4 |
| **Anomaly Detection** | Lambda + statistical + TimesFM | Phase 3 |
| **Outbreak Forecasting** | TimesFM (zero-shot → fine-tuned) | Phase 3 |

All vision outputs tagged: **"AI-Assisted — Requires Doctor Confirmation."**

---

## 4. Data Flow Diagrams

### 4.1 Hackathon POC Data Flow

```
┌─────────────────────────────────────────────────────────┐
│                  HACKATHON POC (Cloud-Only)              │
│                                                         │
│  🧑‍⚕️ User    🎙️ Sarvam API    ☁️ Bedrock     🛡️ Rakshak │
│  speaks  ──→  (ASR/TTS)   ──→  (Claude 4.5)──→ (Rules) │
│              cloud API        JSON extract     triage   │
│                                    │              │     │
│                               🗄️ DynamoDB    ⚠️ Alert   │
│                              (via Lambda)    (if risk)  │
└─────────────────────────────────────────────────────────┘
```

### 4.2 Full Architecture Data Flow (Phase 2+)

```
┌───────────────────────────────────────────────────────┐
│              EDGE LAYER (On-Device)                   │
│                                                       │
│  🧑‍⚕️ ASHA  → 🎙️ Sravana → 📝 Lekhak → 🛡️ Rakshak   │
│  (Voice/    (Sarvam/     (Qwen 3/   (Rule        │
│   Touch/     Whisper)     Bedrock)    Engine)      │
│   Camera)                               │          │
│                                     💾 SQLite      │
│  👁️ Darshak (OCR/Vision) ──────┘        │          │
└──────────────────────────────────────────┼──────────┘
                                           │
                                    📡 Doot (Sync)
                                           │
┌──────────────────────────────────────────┼──────────┐
│              CLOUD LAYER (AWS Mumbai)    │          │
│                                          ▼          │
│  🌐 API GW → ⚡ Lambda → 🗄️ DynamoDB              │
│  + Cognito    FastAPI     🏥 HealthLake (FHIR)     │
│                           📦 S3 (Audio/Images)      │
│                           🇮🇳 ABDM/ABHA Sync        │
└──────────────────────────────────────────┼──────────┘
                                           │
┌──────────────────────────────────────────┼──────────┐
│           INTELLIGENCE LAYER             │          │
│                                          ▼          │
│  📊 QuickSight → 👨‍⚕️ PHC Dashboard                │
│  📈 TimesFM → Outbreak/Resource Forecasting         │
│  🚨 Anomaly Detection → IDSP Alerts                │
│  🔬 Research Data Platform                          │
└─────────────────────────────────────────────────────┘
```

### 4.3 Patient Continuum of Care

```
Visit 1: ASHA Home Visit ──┐
Visit 2: PHC Checkup ──────┼──→ 📋 Unified Patient Record
Visit 3: District Hospital ┤    (via Patient ID / ABHA)
Visit 4: Specialist ────────┘         │
                                      ▼
                            ┌─────────────────────┐
                            │ Complete History:    │
                            │ • All Vitals/Visits  │
                            │ • Medications        │
                            │ • Lab Reports        │
                            │ • Referral Chain     │
                            │ • Imaging Records    │
                            └─────────────────────┘
                                      │
                    ┌─────────────────┼─────────────────┐
                    ▼                 ▼                 ▼
              👨‍⚕️ Any Doctor    🧑 Patient        📊 Research
              (authorized)    (ABHA App)       (anonymized)
```

---

## 5. FHIR Resource Mapping

| Our Data | FHIR R4 Resource | Key Fields | Terminology |
|:---|:---|:---|:---|
| Patient demographics | `Patient` | name, gender, birthDate, identifier (ABHA) | — |
| Vitals (BP, weight, temp) | `Observation` | code, value, effectiveDateTime | LOINC (e.g., BP = 85354-9) |
| Visit record | `Encounter` | status, class, period, participant | — |
| Risk flag / condition | `Condition` | code, severity, onsetDateTime | ICD-10 |
| Referral | `ServiceRequest` | status, intent, code, requester | — |
| Vaccination | `Immunization` | vaccineCode, occurrenceDateTime, status | CVX codes |

**Validation:** HAPI FHIR library for FHIR profile validation before ABDM push. Test in NDHM sandbox before production.

---

## 6. Security & Privacy Architecture

| Layer | Mechanism |
|:---|:---|
| **At Rest (Device)** | SQLite encryption (SQLCipher) |
| **At Rest (Cloud)** | AWS KMS for DynamoDB, S3, HealthLake |
| **In Transit** | TLS 1.3 for all communication |
| **PII Handling** | Patient names → SHA-256 hashed IDs before cloud sync. Real names stored ONLY on-device. |
| **Access Control** | Amazon Cognito: ASHA, MO, DHO, Admin, Researcher roles |
| **Audit Trail** | CloudWatch Logs for all data access/modification |
| **Consent** | Verbal consent recorded as local audio. Digital consent status per patient record. |
| **Data Deletion** | Soft-delete (retained for audit, excluded from analytics). Right to erasure per DPDP Act 2023. |

---

## 7. UX Design Philosophy

### Input Hierarchy
1. **Voice** (Primary) — Hands-free during home visits. Large "बोलिये" button.
2. **Touch** (Secondary) — Corrections, scrolling, confirming.
3. **Camera** (Tertiary) — Scan prescriptions, documents.

### Key UX Patterns

| Pattern | Description |
|:---|:---|
| **Big Button Home** | Minimal text. Pulsing microphone "बोलिये". Offline status banner. |
| **Conversational Forms** | One question at a time via voice. No long scrolling forms. |
| **Verbal Confirmation** | *"Likha hai 2.5 kilo. Sahi hai?"* → "Haan" or correct. |
| **WhatsApp-Style History** | Familiar chat-bubble patient timeline. |
| **Color-Coded Alerts** | 🟢 Normal / 🟡 Monitor / 🔴 Urgent — full-screen color + audio. |
| **Offline Banner** | "✅ Synced" / "📴 Offline — Data Saved Locally" |
| **Error Recovery** | Low confidence → re-prompt. Extraction fail → manual form fallback. |

---

## 8. Integration Architecture

SarvaSukh AI does **not** replace existing apps — it sits on top as an **intelligent voice overlay**.

```
🎙️ Voice / 👆 Touch / 📷 Camera
              │
        ┌─────┴─────┐
        │ SarvaSukh  │ (Voice Overlay Layer)
        │    AI      │
        └─────┬─────┘
              │ Auto-fill via API / screen automation
     ┌────────┼────────┬──────────┐
     ▼        ▼        ▼          ▼
  ANMOL    RCH Portal  NCD App  ABDM/ABHA
```

---

## 9. Scalability & Cost

### AWS Auto-Scaling

| Component | Mechanism |
|:---|:---|
| Lambda | Auto-scales 0 → 10,000 concurrent |
| DynamoDB | On-demand capacity |
| API Gateway | Managed, auto-scales |
| HealthLake | Managed FHIR service |

### Cost Estimates (see Requirements §10 for details)

| Phase | ASHAs | Monthly AWS + API | Per-ASHA Cost |
|:---|:---|:---|:---|
| POC (Hackathon) | Demo | <$10 | N/A |
| Pilot | 100 | $250–$800 | $2.50–$8.00 |
| National | 1,000,000 | $10,000–$30,000 | $0.01–$0.03 |

---

## 10. Hackathon Build Plan (Realistic Scope)

| Sprint | Hours | Deliverable | Skip for Hackathon |
|:---|:---|:---|:---|
| **Sprint 1** | 0–6 | React Native app shell. Integrate Sarvam Saaras V3 API for voice capture + transcription. Basic UI with "बोलिये" button. | |
| **Sprint 2** | 6–12 | Integrate Bedrock Claude 4.5 for entity extraction (JSON mode). Display extracted data in auto-fill form. | On-device Qwen 3 |
| **Sprint 3** | 12–18 | Implement 5 clinical rules (Rakshak). Visual risk alerts (color + audio). Auto-referral summary. | Full rule engine |
| **Sprint 4** | 18–24 | Polish UI. Record 3-minute demo video with 3 scripted scenarios. Prepare pitch. | Offline sync, FHIR, ABDM, TimesFM, Vision |

> **Hackathon Fallback Plan:** If Sarvam API is slow/unavailable → use pre-recorded audio transcripts as mock input. If Bedrock is slow → use hardcoded entity extraction for demo scenarios. The goal is a **compelling demo**, not production-ready code.

---

## 11. Post-Hackathon Roadmap

| Phase | Timeline | Key Milestones |
|:---|:---|:---|
| **Pilot** | Month 1–3 | 10 ASHAs first (fast iteration) → expand to 100. Offline mode. PHC dashboard. ABHA sandbox. |
| **Regional** | Month 4–9 | 10,000 ASHAs, 5 states. WhatsApp integration. Incentive tracking. Multi-lingual expansion. |
| **National** | Month 10–18 | 1M ASHAs. HealthLake data lake. IDSP feed. TimesFM analytics. Government app overlays. |
| **Vision** | Month 18+ | Medical imaging AI. Telemedicine. Health nudges. Research platform. |

---

## 12. Innovation Summary

| Dimension | What's New |
|:---|:---|
| **Architecture** | Multi-agent with specialized roles, not a monolithic chatbot |
| **Connectivity** | Graceful degradation: cloud-first → offline-capable with honest trade-offs |
| **Language** | Indic-native AI (Sarvam) with 22-language code-mixing support |
| **Clinical Safety** | Deterministic rules — no AI hallucination risk for health flags |
| **Data Platform** | Grassroots-up health data lake for national surveillance |
| **Forecasting** | TimesFM for proactive (not reactive) public health |
| **Patient Continuity** | Cross-center tracking via unique ID / ABHA |
| **Integration** | Voice overlay on existing government apps |
| **Hackathon → Product** | Clear POC scope that showcases core value; scalable to national platform |