# SarvaSukh AI — India's Voice-First AI Health Platform

> **AWS AI for Bharat Hackathon** | Track: AI for Healthcare & Life Sciences  
> **Team:** Govind Mittal

---

## 🏥 The Problem

India's **1 million ASHA workers** serve 845 million rural citizens but spend **3–4 hours/day** (40% of work time) on manual data entry across 11–15 paper registers and complex digital forms.

- 48% struggle with app navigation
- \>20% data error rates
- Most areas have 2G or no internet connectivity

## 💡 Our Solution

**SarvaSukh AI** is a voice-first, AI-powered health platform that transforms how ASHA workers collect and process health data:

- **Voice-to-Record:** ASHA speaks in any Indian dialect → AI auto-fills all forms (7 min → 45 sec)
- **Real-time Clinical Triage:** Flags pre-eclampsia, anemia, malnutrition via WHO/Ayushman Bharat rules
- **Offline-Capable:** On-device AI (Qwen 3, Gemma 3) works without internet
- **22 Indian Languages:** Built on Sarvam AI's Indic-native ASR/TTS (Saaras V3 / Bulbul V3)
- **National Health Data Lake:** FHIR-compliant data for disease surveillance & pandemic tracking

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                 EDGE LAYER (On-Device)                   │
│  Sravana (ASR) → Lekhak (NER) → Rakshak (Triage)       │
│  Darshak (Vision/OCR) → SQLite (Local Storage)          │
└────────────────────────┬────────────────────────────────┘
                         │ Doot Agent (Background Sync)
┌────────────────────────▼────────────────────────────────┐
│               CLOUD LAYER (AWS Mumbai)                   │
│  API Gateway + Lambda + Bedrock (Claude 4.5)            │
│  DynamoDB + HealthLake (FHIR R4) + S3 → ABDM/ABHA      │
└────────────────────────┬────────────────────────────────┘
                         │ Data Aggregation
┌────────────────────────▼────────────────────────────────┐
│              INTELLIGENCE LAYER                          │
│  QuickSight Dashboards │ TimesFM Forecasting │ IDSP     │
└─────────────────────────────────────────────────────────┘
```

## 🤖 Multi-Agent System

| Agent | Role | Technology |
|-------|------|-----------|
| **Sravana** | Voice-to-Text | Sarvam Saaras V3 (22 languages) |
| **Lekhak** | Form Auto-Fill | Qwen 3 0.5B (on-device NER) |
| **Rakshak** | Clinical Triage | Deterministic WHO/AB rules |
| **Darshak** | Vision & OCR | Qwen 3 VL / Claude 4.5 Vision |
| **Doot** | Sync Agent | Background sync with exponential backoff |

## 🛠️ Tech Stack

**Edge / Offline:**
- Qwen 3 0.5B (4-bit GGUF, llama.cpp)
- Sarvam AI Saaras V3 / Bulbul V3
- SQLite (WAL mode), React Native (Android)

**AWS Cloud:**
- Amazon Bedrock (Claude 4.5), Lambda, API Gateway
- DynamoDB, HealthLake (FHIR R4), S3, Cognito
- QuickSight, SNS/SQS, CloudWatch

**Analytics:**
- Google TimesFM (outbreak/resource forecasting)
- IDSP automated surveillance alerts

## 📊 Impact

| Metric | Before | After |
|--------|--------|-------|
| Form fill time | 7 min | 45 sec |
| Daily data entry | 4 hrs | <30 min |
| Data error rate | >20% | <5% |
| Outbreak detection | Weeks | <24 hrs |

> **Returns 3 million healthcare hours to rural India every day.**

## 🇮🇳 National Mission Alignment

- Ayushman Bharat Digital Mission (FHIR + ABHA ID)
- National Health Mission (1M ASHAs)
- IDSP Disease Surveillance
- POSHAN Abhiyaan
- Mission Indradhanush
- India AI Mission

## 📂 Repository Structure

```
├── README.md                              # This file
├── requirements.md                        # Detailed project requirements
├── design.md                              # System design document
├── create_presentation.py                 # Presentation generation script
├── SarvaSukh_AI_Idea_Submission.pptx      # Final presentation (11 slides)
└── .gitignore
```

## 💰 Cost Estimate

| Phase | Cost |
|-------|------|
| Hackathon POC | < $10 |
| Pilot (100 ASHAs, 3 mo) | $250–800/mo |
| National Scale (1M ASHAs) | $0.01–0.03/ASHA/mo |

> **90%+ operations happen on-device → minimal cloud costs**

---

*Built for the AWS AI for Bharat Hackathon — Powered by AWS*
