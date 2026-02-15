# Project Name: SarvaSukh AI
**Tagline:** India's Voice-First, Offline-Ready AI Health Platform — From ASHA Worker to Policy Maker.

**Track:** AI for Healthcare & Life Sciences  
**Team:** Govind Mittal (Solo)

---

## 1. Problem Statement

### The Crisis: India's Healthcare Last Mile is Drowning in Paperwork

India's primary healthcare system rests on the shoulders of **1 million ASHA (Accredited Social Health Activist) workers** who serve **845 million rural Indians**. Each ASHA covers ~1,000 people, acting as the sole bridge between rural communities and the formal health system. Despite contributing to a historic MMR decline from 384 → 93 per lakh live births (2000–2021), they are systematically failed by the tools they are given.

### Pain Point 1: The Data Entry Burden (Time Poverty)
- ASHA workers spend **3–4 hours daily** (~40% of their work time) manually filling **11–15 paper registers** and complex digital forms *(Source: PMC Time Motion Study, 2024)*.
- Current government apps (ANMOL, NCD App, RCH Portal) require extensive **typing** and English/Formal Hindi literacy.
- **48%** of ASHA workers struggle with app navigation; data error rates **>20%**.

### Pain Point 2: The Digital Divide (Usability Failure)
- Apps are designed for urban, literate users. Rural ASHAs are comfortable with **WhatsApp voice notes**, not form-based interfaces.
- Poor internet connectivity (2G/Offline) in many rural areas renders cloud-dependent apps useless.
- OTP-based verification and location tracking features create distrust.

### Pain Point 3: The Silent Clinical Gap
- Critical warning signs (BP 140/90 in pregnant woman, child <2kg) are **recorded but not acted upon** — no automated triage exists.
- IDSP disease surveillance still relies on **paper-based S/P/L forms** — outbreaks detected days/weeks late.
- Eclampsia, severe anemia, and malnutrition go undetected despite being recorded.

### Pain Point 4: The Data Silo Problem
- Health data is fragmented across paper registers, state-specific apps, and district databases.
- No unified patient health record across visits to different centers/doctors.
- ABDM envisions interoperable records via ABHA IDs, but ground-level adoption lags severely.

### The Opportunity
By automating clerical work through Voice AI, we can return **3 million hours of healthcare to rural India every day**. By adding clinical intelligence, we can prevent preventable deaths. By centralizing grassroots data, we build India's most powerful public health surveillance platform.

---

## 2. Target Audience & Personas

| Persona | Who | Need | Pain |
|:---|:---|:---|:---|
| **Sarita** (ASHA Worker) | Female, 25–45, Class 10–12, rural, $80 Android phone, 2GB RAM | Speak → auto-fill forms. Finish paperwork in 10 min. | 3–4 hrs/day data entry. Late pay from claim rejections. |
| **Dr. Verma** (PHC Medical Officer) | Primary Health Centre doctor | Real-time dashboard of high-risk patients. Accurate referrals. | Receives incomplete/late referrals. Cannot prioritize. |
| **Block Health Officer** | District/Block administration | Aggregated data for resource allocation and outbreak detection. | Data arrives weeks late, on paper. |
| **Policy Maker / Researcher** | NHM, ICMR, WHO | Anonymized population-scale health data for policy and research. | Paper-based, inaccessible data. |

**Integration Philosophy:** SarvaSukh AI sits **on top of existing apps** (ANMOL, RCH, NCD) as an intelligent voice overlay — it does not replace them, it makes interactions seamless and voice-driven.

---

## 3. Proposed Solution: SarvaSukh AI

**SarvaSukh AI** is a **voice-first, offline-capable, multi-agent AI health platform** that transforms how India's frontline health workers collect data, detect health risks, and contribute to a national health intelligence system.

### Core Value Proposition
1. **For ASHA Workers:** Speak naturally in any Indian dialect → AI auto-fills all forms (7 min → 45 sec). No typing. No English needed.
2. **For Medical Officers:** Auto-prioritized high-risk patient dashboards with referral tracking.
3. **For the Nation:** A grassroots health data lake feeding disease surveillance, pandemic tracking, and evidence-based policy.

### What Makes SarvaSukh AI Different (USP)

| Differentiator | Description |
|:---|:---|
| **Offline-Capable** | On-device AI (Qwen 3 / Gemma 3 quantized) handles entity extraction. Cloud AI used when online for complex cases. Graceful degradation, not hard offline-only. |
| **Multi-Agent, Not Chatbot** | 5 specialized agents (Sravana, Lekhak, Rakshak, Darshak, Doot) with deterministic clinical guardrails. |
| **Indic-Native AI** | Built on Sarvam AI Saaras V3 (ASR, 22 languages, code-mixing) and Bulbul V3 (TTS, 11 languages). Not a translated Western model. |
| **Safety-First** | Clinical flags use deterministic Python rules from WHO/Ayushman Bharat guidelines. AI **never diagnoses or prescribes** — only recommends referrals. |
| **Integration Overlay** | Sits on top of ANMOL/RCH/NCD as a voice layer. Zero learning curve for new apps. |
| **Patient Continuity** | Unique Patient ID linked to ABHA. Complete health history across all visits, centers, doctors. |

---

## 4. Functional Requirements

### Phase 1: Hackathon POC (Proof of Concept — Build Sprint)

> **Hackathon Scope:** A working demo that showcases the core voice-to-record and clinical triage workflow. Uses cloud APIs (Sarvam, Bedrock) — on-device optimization is a post-hackathon goal.

#### F1. Voice-to-Record Demo ("Lekhak" Agent)
- **Input:** User speaks in Hindi (expandable to other languages post-hackathon).
- **Processing:** Sarvam AI Saaras V3 (cloud API) transcribes → Amazon Bedrock (Claude 4.5) extracts structured entities in JSON mode.
- **Entities Extracted:** Patient Name, Age, Gender, Vitals (BP, Weight, Temperature), Symptoms, Visit Type (ANC/PNC/Immunization/NCD), Date.
- **Output:** Auto-populated form. Verbal confirmation via Sarvam Bulbul V3 TTS: *"Maine likha: Sunita, BP 150/90, chakkar. Sahi hai?"*
- **Demo Scenarios:** 3–5 scripted patient interactions covering ANC visit, immunization, NCD screening.
- **Accuracy Target:** >95% entity extraction on demo scenarios.

#### F2. Clinical Triage Demo ("Rakshak" Agent)
- **MVP Rule Set (5 Critical Rules):**

| # | Condition | Rule | Alert Level |
|:--|:---|:---|:---|
| 1 | Pre-eclampsia Risk | `BP_Sys > 140 AND Pregnant == True` | 🔴 CRITICAL |
| 2 | Severe Low Birth Weight | `Child_Weight < 2.0 kg` | 🔴 CRITICAL |
| 3 | Severe Anemia | `Hb < 7 g/dL` | 🔴 CRITICAL |
| 4 | Suspected Measles (IDSP) | `Temp > 103°F AND Rash == True` | 🟡 REPORT |
| 5 | Missed Vaccination | `Child_Age > Due_Date AND Vaccine_Not_Given` | 🟢 REMINDER |

- **Output:** Visual alert (screen color change), audio alert in Hindi, auto-generated referral summary.
- **Safety:** AI never diagnoses or prescribes. Only flags risks based on published guidelines and recommends referrals.

#### F3. Basic UI
- Large "बोलिये" (Speak) button — voice-first interaction.
- Form auto-fill display with edit capability.
- Patient history view (WhatsApp-style chat timeline).
- Color-coded risk alerts (Green/Yellow/Red).

#### F4. Cloud Architecture (Hackathon)
- Frontend → API Gateway → Lambda (FastAPI) → DynamoDB.
- Sarvam API for ASR/TTS. Bedrock for entity extraction.
- No offline mode in POC (uses cloud APIs).

---

### Phase 2: Pilot (Post-Hackathon — 100 ASHAs, 1 District)

#### F5. On-Device AI for Offline Capability
- **ASR:** Sarvam Saaras V3 cloud API (primary). Fallback: Whisper Tiny (39M params, quantized INT8, ~100MB) for offline ASR. *Trade-off: lower accuracy for dialect/code-mixing, but functional offline.*
- **Entity Extraction:** Qwen 3 0.5B (4-bit GGUF, ~250MB, via llama.cpp). Realistic latency on 2GB RAM device: **5–10 seconds** (CPU-only, no GPU). Acceptable for offline use; complex cases queued for cloud processing when online.
- **Clinical Rules:** Python rule engine — always on-device, sub-second latency.
- **Storage:** SQLite (WAL mode). ~50MB for 6 months of patient data per ASHA.
- **Sync:** Background sync with exponential backoff. Conflict resolution: timestamp + device ID; critical field conflicts surface in PHC "Review Needed" queue. PHC doctor's edits always take precedence.

#### F6. Expanded Clinical Rule Set (15 Rules)
- Add: Gestational Diabetes, Suspected TB, Severe Acute Malnutrition (SAM), Postpartum Hemorrhage risk, Neonatal danger signs, NCD screening thresholds (Hypertension, Diabetes).
- **Rule Versioning:** Rules stored as JSON config file, fetched from cloud weekly, cached locally. Version-stamped. No app update needed for rule changes.
- **Priority System:** CRITICAL (immediate) > HIGH (same day) > MODERATE (this week) > LOW (routine). Multiple alerts show in priority order.

#### F7. PHC Medical Officer Dashboard
- Real-time high-risk patient lists (auto-prioritized by severity).
- Visit tracking, referral status management.
- Conflict resolution queue for sync conflicts.
- Powered by Amazon QuickSight.

#### F8. ABHA / ABDM Integration (Minimal Viable)
- **FHIR Resources (MVP):** `Patient`, `Observation` (vitals), `Encounter` (visit), `Condition` (diagnosis/risk flag).
- **Terminology:** LOINC codes for vitals (e.g., BP → 85354-9), ICD-10 for conditions.
- **ABDM Sandbox:** Test with NDHM FHIR profiles before production.
- **Fallback ID:** If no ABHA → generate local UUID, link to ABHA later.

#### F9. Automated Incentive Tracking
- Auto-calculate expected ASHA payment based on logged work.
- Flag discrepancies with government payment records.
- Target: Claim rejection rate from **15% → <1%**.

---

### Phase 3: Scale (10,000 ASHAs, 5 States)

#### F10. Multi-Lingual Expansion
- **22 languages** via Sarvam Saaras V3 (cloud). Offline ASR coverage expanded as Sarvam/community releases on-device models.
- **11 TTS languages** via Sarvam Bulbul V3.
- Priority: Hindi, Tamil, Telugu, Bengali, Marathi, Rajasthani.

#### F11. Centralized Health Data Lake
- AWS HealthLake (FHIR R4) for aggregated, anonymized health data.
- Integration with ICMR, WHO for epidemiological research.
- Data governance: DPDP Act 2023 compliant, audit-logged, consent-based.

#### F12. Disease Surveillance & Pandemic Tracking
- Automated IDSP reporting (replacing paper S/P/L forms).
- Anomaly detection: geographic clustering of symptoms → outbreak alert.
- Escalation: village → block → district → state → national.

#### F13. Time Series Health Analytics (Vishleshan Engine)
- **Model:** TimesFM (Google, open-source, pre-trained on 100B time points).
- **Input:** Daily aggregated metrics per district — fever incidence, ANC visits, vaccination counts, vitals distributions.
- **Forecasting:** Zero-shot initially; fine-tune on IDSP historical data if zero-shot accuracy <80%.
- **Horizon:** 7-day (outbreak alerts), 30-day (resource planning), 90-day (seasonal patterns).
- **Use Cases:** Dengue/malaria surge prediction, medicine stock-out forecasting, hospital bed occupancy.
- **Validation:** Backtest on 2 years of historical IDSP data before deployment.

#### F14. WhatsApp Integration
- Referral tickets to PHC doctor, vaccination/ANC reminders to beneficiaries.
- Via Sarvam Samvaad + WhatsApp Business API.

#### F15. Government App Integration Layer
- Voice overlay plugin for ANMOL, RCH Portal, NCD App.
- ASHA speaks → SarvaSukh fills the underlying app's forms.

---

### Phase 4: Vision & Future (1M ASHAs, National)

#### F16. Medical Imaging AI ("Darshak" Vision Capabilities)
- **OCR + Document Understanding:** Qwen 3 VL (on-device, basic). Claude 4.5 Vision (cloud, complex).
- **Medical Images:** X-ray/ECG photograph → preliminary AI reading → "AI-Assisted — Requires Doctor Confirmation."
- **Health Record Digitization:** Scan handwritten records/prescriptions → structured data.

#### F17. Telemedicine Escalation
- Red Flag case → live video call with specialist via eSanjeevani.
- AI provides pre-summarized patient brief to doctor.

#### F18. Personalized Health Nudges
- SMS/WhatsApp: Vaccination schedules, ANC reminders, medication adherence (TB DOTS, hypertension, diabetes), nutrition advice.

---

## 5. Non-Functional Requirements

| Category | Requirement | POC (Hackathon) | Pilot (Phase 2) | Scale (Phase 3+) |
|:---|:---|:---|:---|:---|
| **Latency** | Voice-to-Structured Data | <5 sec (cloud) | <10 sec (offline) / <5 sec (online) | <5 sec (online) |
| **Accuracy** | Vital Extraction | >95% (demo) | >95% (online) / >90% (offline) | >98% (online) |
| **Device** | Minimum Spec | Any Android | $80, 2GB RAM, Android 10+ | Same |
| **Connectivity** | Offline Support | ❌ Cloud-only | ✅ Graceful degradation | ✅ Full |
| **Availability** | Cloud Uptime | Best effort | 99% | 99.5% |
| **Data** | Residency | AWS Mumbai (ap-south-1) | Same | Same |
| **Privacy** | PII Protection | N/A (demo data) | Hashed IDs before cloud sync | Full DPDP compliance |
| **Clinical Safety** | Guardrail | AI never prescribes | Same + rule versioning | Same + audited rules |

---

## 6. Error Handling & Edge Cases

| Scenario | Handling |
|:---|:---|
| ASR returns gibberish (noisy environment) | Confidence score < 0.6 → prompt re-speak: *"Sunai nahi diya, phir se boliye"* |
| Entity extraction fails (ambiguous input) | Highlight uncertain fields in yellow → ask ASHA to confirm/correct verbally or by touch |
| Patient has no ABHA ID | Generate local UUID → link to ABHA when patient registers |
| Multiple alerts triggered simultaneously | Display in priority order (CRITICAL first). All alerts shown, not just top one. |
| Rule engine encounters missing data | Flag as "Incomplete Data" → do not suppress alerts, ask ASHA for missing vitals |
| Sync fails repeatedly | After 5 retries, surface "Sync Failed" notification → data safe in local SQLite |
| Two ASHAs record conflicting data for same patient | Timestamp + device ID conflict detection → "Review Needed" in PHC queue |

---

## 7. User Training & Onboarding

| Phase | Training Approach |
|:---|:---|
| **First Launch** | In-app guided tutorial (voice-guided, <5 min): "Press the big button and speak" |
| **Pilot** | 1-day in-person training at sub-centre. Buddy system (trained ASHA helps new user). |
| **Support** | WhatsApp helpline for issues. FAQ bot in local language. |
| **Updates** | App via Google Play. Clinical rules via cloud config (no app update needed). |

---

## 8. Privacy & Consent

| Aspect | Mechanism |
|:---|:---|
| **Consent Capture** | Verbal consent recorded as audio file (stored locally, not synced). Digital record of consent status per patient. |
| **Consent for Minors** | Parent/guardian consent captured verbally. Guardian name + relationship recorded. |
| **Consent Withdrawal** | Patient can request withdrawal via ASHA → record marked "Consent Withdrawn" → data soft-deleted (retained for audit, not used for analytics). |
| **Data Retention** | Active records: indefinite. Inactive (no visit in 3 years): anonymized. Withdrawn: soft-deleted. |
| **Research Data** | De-identified (k-anonymity). Ethics committee approval required. Granular opt-in/opt-out. |

---

## 9. Success Metrics (SMART)

### Hackathon POC Success
| Metric | Target |
|:---|:---|
| Demo scenarios completed | 3–5 scripted patient interactions working end-to-end |
| Entity extraction accuracy | >95% on demo scenarios |
| Clinical triage alerts | All 5 rules trigger correctly on test data |
| Demo video | 3-minute video showing voice-to-form + triage alert flow |

### Pilot Success (100 ASHAs, 3 months)

| Metric | Baseline (Pre-Pilot) | Target | Measurement |
|:---|:---|:---|:---|
| Daily data entry time | 3–4 hours (survey baseline) | **<1.5 hours** (>50% reduction) | Self-reported time diary + app usage logs |
| Data entry error rate | >20% (cross-validation with supervisor) | **<5%** | Supervisor spot-checks on 10% of records |
| ASHA adoption rate | 0% (new tool) | **>80%** using app ≥3 days/week | App usage analytics |
| High-risk referrals generated | Baseline count in pilot district | **+30% increase** | Referral slip count comparison |
| Incentive claim rejection | ~15% (district records) | **<5%** | Payment records comparison |
| User satisfaction (NPS) | N/A | **NPS > 40** | Monthly survey |

### Health Outcome Indicators (Long-term Monitoring)

| Indicator | Current (NFHS-5) | Expected Direction |
|:---|:---|:---|
| High-risk pregnancy detection | Under-detected | ↑ Earlier detection via automated triage |
| Severe anemia (Hb<7) in pregnant women | 52.2% anemic overall | ↑ Faster identification → timely treatment |
| SAM identification in under-5s | 35.5% stunted, 19.3% wasted | ↑ Real-time weight tracking → earlier NRC referral |
| Full vaccination coverage (12-23 mo) | 76.4% | ↑ Automated scheduling + reminders |
| NCD screening coverage | 24% men / 21% women hypertensive, low screening | ↑ Simplified voice-based recording |
| Disease outbreak detection time | Days–weeks (paper IDSP) | ↓ Target: <24 hours |

---

## 10. Cost Estimates (Realistic)

### Hackathon Phase
- **AWS:** Free tier (Lambda, DynamoDB, API Gateway) — **~$0** for demo usage.
- **Sarvam AI:** Developer/trial API access — confirm pricing with Sarvam team.
- **Bedrock (Claude 4.5):** ~$0.50–$2 for demo volume (~100 calls).
- **Total:** **<$10** for hackathon.

### Pilot Phase (100 ASHAs, 3 months)

| Service | Monthly Estimate | Notes |
|:---|:---|:---|
| Lambda + API Gateway | $20 | Low request volume, mostly free tier |
| DynamoDB (on-demand) | $50 | ~100K reads/writes per month |
| Bedrock (Claude 4.5) | $100–$500 | **Depends on cloud escalation rate** (see below) |
| S3 | $10 | Audio files, scanned documents |
| Sarvam AI API | $50–$200 | **Requires confirmation with Sarvam** |
| CloudWatch + Cognito | $20 | Monitoring, auth |
| **Total** | **$250–$800/month** | |

> **Cost Sensitivity:** If 5% of requests escalate to Bedrock → ~$100/mo. If 20% escalate → ~$500/mo. On-device processing (Qwen 3 offline) reduces cloud costs significantly. Key variable is Sarvam API pricing — needs negotiation for pilot tier.

### National Scale (1M ASHAs) — Rough Estimate
- **$10,000–$30,000/month** depending on cloud escalation rates, Sarvam enterprise pricing, HealthLake volume.
- **Per-ASHA cost:** ~$0.01–$0.03/month at scale — economically viable for government program.
- **Note:** 90%+ of operations happen on-device at scale, minimizing cloud costs.

---

## 11. Regulatory & Compliance Considerations

| Area | Status / Plan |
|:---|:---|
| **DPDP Act 2023** | Data minimization, consent-based processing, right to erasure (soft-delete) |
| **CDSCO (Medical Device)** | SarvaSukh is a **clinical decision support tool**, not a diagnostic device. Does not require CDSCO approval as it only flags risks, never diagnoses. Legal review recommended before pilot. |
| **Ethics Committee** | Required for pilot. Will submit protocol to district ethics committee. |
| **ABDM Compliance** | FHIR R4 format, ABHA ID integration, NDHM profile validation in sandbox |
| **Liability** | System is advisory only. All alerts include "AI-Assisted — Consult Doctor." ASHA/doctor makes final decision. |

---

## 12. Alignment with National Missions

| Mission | How SarvaSukh AI Aligns |
|:---|:---|
| **Ayushman Bharat Digital Mission** | FHIR-compliant records, ABHA ID integration |
| **National Health Mission** | Directly empowers ASHAs, improves MCH indicators |
| **IDSP** | Automated real-time disease reporting |
| **POSHAN Abhiyaan** | Malnutrition tracking, early NRC referrals |
| **Mission Indradhanush** | Vaccination tracking & reminders |
| **India AI Mission** | Sovereign Indic AI, on-device inference |
