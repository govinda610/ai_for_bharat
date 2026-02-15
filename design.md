# System Design: SarvaSukh Sahayak
**Architecture Pattern:** Hybrid Edge-Cloud Multi-Agent System.

---

## 1. High-Level Architecture
The solution uses a **"Local-First, Cloud-Sync"** approach. The heavy lifting of understanding the ASHA worker is done on the device (Edge) to ensure functionality in rural areas, while the Cloud is used for heavy model training, analytics, and long-term storage.

### The Stack
* **Frontend:** React Native / Android Native (Optimized for low-end devices).
* **Voice Layer (ASR):** **Sarvam AI / Bhashini APIs** (for high-fidelity Indic language support) + **OpenAI Whisper Tiny** (Fallback for offline English/Hinglish).
* **Intelligence Layer (The Brain):**
    * **Edge:** Quantized SLM (Small Language Model) like **Llama-3-8B (4-bit)** or **Gemma-2-2B** running via MLC LLM.
    * **Cloud:** AWS Bedrock (Claude 3 Haiku) for complex query resolution and periodic syncing.
* **Backend:** FastAPI (Python) hosted on AWS Lambda (Serverless).
* **Database:** SQLite (On-Device) + AWS DynamoDB (Cloud).

---

## 2. Agentic Workflow (The "Brain" Logic)
The system is not just a chatbot; it is a team of specialized AI agents working together.

### Step 1: The "Sravana" (Listener) Agent
* **Trigger:** ASHA worker presses the "Mic" button.
* **Task:** Captures raw audio, performs noise cancellation, and transcribes local dialect to standardized text.
* **Tech:** Sarvam AI API / On-device Whisper.

### Step 2: The "Lekhak" (Scribe) Agent
* **Task:** Structured Entity Extraction (NER).
* **Process:**
    * *Input Text:* "Sunita ka BP 150/90 hai aur usko chakkar aa rahe hain."
    * *Extraction:* `{ "Patient": "Sunita", "Vitals": {"BP_Sys": 150, "BP_Dia": 90}, "Symptoms": ["Dizziness"] }`
* **Tech:** On-Device SLM (JSON Mode).

### Step 3: The "Rakshak" (Guardian) Agent
* **Task:** Clinical Validation & Triage.
* **Logic:**
    * Runs deterministic Python rules (Non-AI) against the extracted JSON.
    * `IF BP_Sys > 140 -> RETURN "High Risk: Pre-eclampsia Alert"`
* **Output:** UI turns RED, plays a warning sound, and prompts for "Referral."

### Step 4: The Sync Agent
* **Task:** Data transmission.
* **Logic:** Checks network status. If Online -> Push to FHIR Server / Government Portal. If Offline -> Store in local SQLite and retry later.

---

## 3. Data Flow Diagram
```mermaid
graph TD
    User[ASHA Worker] -->|Voice Command| App[Mobile App]
    subgraph "Edge Layer (On-Device)"
        App -->|Raw Audio| ASR[Speech-to-Text Engine]
        ASR -->|Text| SLM[Small Language Model (Llama-3 Quantized)]
        SLM -->|JSON Data| RuleEngine[Clinical Rule Engine]
        RuleEngine -->|Validation| LocalDB[(SQLite Local Storage)]
    end
    subgraph "Cloud Layer (AWS)"
        LocalDB -->|Sync (When Online)| CloudAPI[FastAPI Gateway]
        CloudAPI -->|Store| CloudDB[(DynamoDB/HealthLake)]
        CloudAPI -->|Analytics| Dashboard[MO Dashboard]
    end
    RuleEngine -->|Risk Alert| App

4. User Experience (UX) Design
Philosophy: "Zero-Typing Interface."

The "Big Button" Home: The home screen has minimal text. It features a large, central microphone button labeled "Boliye" (Speak).

Conversational Forms: Instead of a long scrolling list, the app asks one question at a time verbally: "What is the baby's weight?"

Visual Confirmation:

The app repeats the data back: "Maine likha hai 2.5 kilo. Sahi hai?" (I wrote 2.5kg. Is this correct?)

User says "Haan" (Yes) or "Nahi, 3 kilo" (No, 3kg).

Whatsapp-Style History: Patient records look like a chat history, which is a familiar interface for rural users.

5. Implementation Plan (Hackathon Phase)
Sprint 1 (Hours 0-6): Setup Basic App UI & Integrate Sarvam/Bhashini Voice API.

Sprint 2 (Hours 6-12): Build the "Lekhak Agent" (Prompt Engineering Llama-3 to output JSON from medical transcripts).

Sprint 3 (Hours 12-18): Implement the "Rakshak Agent" (Rule Engine) and connect it to a dummy form.

Sprint 4 (Hours 18-24): Optimize for Offline Mode (Mocking the edge behavior) and creating the Pitch Demo Video.

6. Future Scope
Integration with ABHA: Linking records to the Ayushman Bharat Health Account ID.

Telemedicine Upgrade: Allowing the ASHA worker to escalate a "Red Flag" case to a video call with a doctor within the same app.