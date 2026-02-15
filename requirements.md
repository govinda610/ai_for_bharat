# Project Name: SarvaSukh Sahayak
**Tagline:** The Offline-First, Voice-Native AI Co-Pilot for India’s 1 Million ASHA Workers.

---

## 1. Problem Statement
**The Crisis:** "ASHA workers are turning into Data Entry Operators, and rural healthcare is paying the price."

India’s primary healthcare relies on **1 million ASHA (Accredited Social Health Activist) workers**. While they are meant to be community health guardians, they are currently overburdened by a broken digital workflow.

### Key Pain Points (Evidence-Backed)
1.  **The Data Burden (Time Poverty):**
    * Research shows ASHA workers spend **3-4 hours daily** (approx. 40% of their work time) manually filling up to **15 different physical registers** and complex digital forms (Source: *PMC Time Motion Study, 2024*).
    * This is time *stolen* from patient care (home visits, counseling, and antenatal checkups).
2.  **The Digital Divide (Usability Failure):**
    * Current government apps (e.g., ANMOL, NCD) require extensive **typing** and English/Formal Hindi literacy.
    * **48%** of ASHA workers struggle with app navigation, leading to data error rates of **>20%** or complete non-usage.
3.  **The "Silent" Clinical Gap:**
    * Data is currently treated as "bureaucracy," not "intelligence." Critical warning signs (e.g., a pregnant woman with BP 140/90) are often recorded in a register but not acted upon until it is too late (e.g., eclampsia).

### The Opportunity (Antyodaya Mission)
By automating the "clerical" work through Voice AI, we can return **3 million hours of healthcare** to rural India every single day, while adding a safety layer of "Clinical Intelligence" to prevent preventable deaths.

---

## 2. Target Audience & Personas
* **Primary User:** **Sarita (The ASHA Worker).**
    * *Context:* Low digital literacy, comfortable with WhatsApp voice notes, works in low-connectivity (2G/Offline) zones.
    * *Need:* "I want to speak, not type. I want to be paid on time."
* **Secondary User:** **Dr. Verma (PHC Medical Officer).**
    * *Need:* Accurate, real-time data to prioritize high-risk patients before they arrive at the hospital.

---

## 3. Functional Requirements
The system is designed as a **Multi-Agentic AI Co-Pilot** that functions as a "Voice Overlay" on top of existing workflows.

### A. Core Features (The MVP)
1.  **Voice-to-Record (The "Scribe" Agent):**
    * **Input:** User speaks in local dialects (Hindi, Rajasthani, Marathi, etc.).
    * **Action:** The AI extracts medical entities (Patient Name, Vitals, Symptoms, Medicines).
    * **Output:** Auto-fills the fields in the structured digital form (JSON/Government App Schema).
2.  **Offline-First Architecture:**
    * System must perform Speech-to-Text (STT) and Entity Extraction **locally on the device** or queue data to sync when the network returns.
    * *Constraint:* No dependency on "always-on" cloud APIs for core data entry.
3.  **Real-Time Clinical Triage (The "Guardian" Agent):**
    * **Logic:** Compares extracted vitals against **Ayushman Bharat / WHO Guidelines**.
    * **Alert:** If a value is "High Risk" (e.g., Child Weight < 2kg), the AI triggers an immediate audio visual alert: *"⚠️ Risk Detected: Severe Malnutrition. Refer to PHC."*
4.  **Multi-Modal Input:**
    * Support for scanning handwritten prescriptions or past records via camera (OCR) to reduce redundancy.

### B. Scalability Features (Phase 2)
* **Automated Incentive Tracking:** Auto-calculate the ASHA's expected payment based on the work logged to reduce claim rejections.
* **WhatsApp Integration:** Send patient referral tickets directly to the PHC Doctor via WhatsApp API.

---

## 4. Non-Functional Requirements (Technical Constraints)
* **Latency:** Voice-to-Text processing must occur within **<3 seconds** to maintain conversational flow.
* **Accuracy:**
    * Entity Extraction (Vitals/Names): **>98% accuracy**.
    * Intent Recognition: **>95% accuracy**.
* **Resource Efficiency:** App must run on low-end Android devices ($80 smartphones) without draining the battery.
* **Privacy & Safety:**
    * **PII Redaction:** Patient names are anonymized before any cloud sync for analytics.
    * **No "AI Doctor":** The AI never prescribes medicine; it only *flags risks* based on deterministic rules (Safety Guardrails).

---

## 5. Success Metrics (KPIs)
| Metric Category | Metric Name | Target |
| :--- | :--- | :--- |
| **Operational** | Time to fill "Antenatal Care Form" | Reduce from **7 mins** (Manual) to **45 secs** (Voice). |
| **Financial** | Incentive Claim Rejection Rate | Reduce from **15%** to **<1%** (via validated data). |
| **Clinical** | High-Risk Pregnancy Detection | Increase identification by **30%** via automated triage rules. |
