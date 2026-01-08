# Book Expert: The AI Audio Experience Platform

**Book Expert** is an enterprise-grade, event-driven platform that transforms static PDF documents into immersive, cinematic audio experiences. Unlike traditional Text-to-Speech (TTS) converters, Book Expert leverages a sophisticated microservices architecture to analyze content, generate emotive narration, and weave in dynamic soundscapes, creating a "movie for your ears."

## 🚀 Key Value Proposition

*   **Cinematic Immersion:** Goes beyond flat narration by integrating background music and sound effects tailored to the text's mood.
*   **Scalable Architecture:** Built on **NATS JetStream**, the system decouples processing stages (OCR, TTS, Mixing), allowing infinite horizontal scaling to handle heavy workloads.
*   **Pay-per-Job Model:** Precise cost estimation based on character count and selected enhancements ensures transparent, usage-based billing.
*   **Enterprise Security:** Self-hosted capabilities with strict data isolation using NATS Object Stores and PostgreSQL multi-tenancy.

---

## 🏗️ System Architecture

The platform is designed as a pipeline of specialized, autonomous microservices.

```mermaid
graph TD
    subgraph "Frontend Layer"
        UI[UI Service (React/Go)] -->|NATS Request| Estimate[Cost Estimator]
        UI -->|Upload| Obj[NATS Object Store]
    end

    subgraph "Processing Core"
        PDF[PDF Processor] -->|Events| OCR[OCR Service]
        OCR -->|Events| TTS[TTS Service]
        TTS -->|Events| Audio[Audio Server (AI/Python)]
        TTS -->|Events| Mix[Soundscape Mixer]
    end

    subgraph "Data & State"
        NATS[(NATS JetStream)]
        PG[(PostgreSQL)]
    end

    UI --> NATS
    Estimate --> NATS
    PDF --> NATS
    OCR --> NATS
    TTS --> NATS
    Mix --> NATS
```

### Core Services

1.  **UI Service (BFF):** A robust Go backend serving a modern React/Vite frontend. Handles authentication, file uploads, and real-time WebSocket progress updates.
2.  **Cost Estimator:** A specialized worker that analyzes document density (character count vs. page count) to provide precise, real-time pricing quotes before processing begins.
3.  **PDF-to-PNG & PNG-to-Text:** High-fidelity document ingestion pipeline using Ghostscript and Vision AI to preserve layout context during text extraction.
4.  **TTS Service:** The orchestration engine for speech generation, managing workload distribution across GPU-accelerated workers.
5.  **Audio Server (Python):** A dedicated AI inference engine hosting state-of-the-art Voice Cloning and Neural Audio models (VoxCPM, etc.).
6.  **Soundscape Service:** The "Director" of the audio experience. It analyzes text sentiment and mixes narration with ambient music and sound effects using `ffmpeg` and `sox`.

---

## ⚡ Technology Stack

*   **Communication:** NATS JetStream (Streams, KV, Object Store) for ultra-low latency, persistent messaging.
*   **Backend:** Go (Golang) 1.24+ for high-concurrency microservices.
*   **AI/ML:** Python 3.12+ with PyTorch and CUDA for neural inference.
*   **Frontend:** React 18, TypeScript, Vite, Material UI (MUI) for a responsive, dark-mode "Studio" experience.
*   **Infrastructure:** Docker-ready, Linux-native optimizations.

## 🔄 The Event-Driven Workflow

1.  **Ingestion:** User uploads a PDF. The **UI Service** streams it to the **NATS Object Store** (`PDFS` bucket).
2.  **Estimation:** The **Cost Estimator** fetches the file, calculates character density, and returns a binding quote.
3.  **Payment & Trigger:** Upon user approval, a `JobSubmitted` event fires.
4.  **Decomposition:** The **PDF Service** explodes the document into high-res images to capture visual context.
5.  **Extraction:** The **OCR Service** extracts text, filtering out artifacts (headers/footers).
6.  **Performance:** The **TTS Service** requests audio generation from the **Audio Server**, chunk by chunk.
7.  **Production:** The **Soundscape Service** aggregates the audio chunks, overlays the selected musical theme, and renders the final `.wav`/`.mp3` artifact.

## 🛡️ Operational Excellence

*   **Explicit State:** No magic. Pricing and features are explicitly toggled by the user.
*   **Resilience:** Every service is stateless and idempotent. If a worker dies, NATS redelivers the message to another instance instantly.
*   **Observability:** Structured JSON logging and centralized state management via NATS Key-Value buckets.

---

*Built with ❤️, Craftsmanship, and Discipline.*
