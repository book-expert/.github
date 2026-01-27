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
        UI["UI Service (React/Go)"] -->|NATS Request| Estimate["Cost Estimator"]
        UI -->|Upload| Obj["NATS Object Store"]
    end

    subgraph "Processing Core"
        PDF["PDF Processor"] -->|Events| OCR["OCR Service"]
        OCR -->|Events| TTS["TTS Service"]
        TTS -->|Events| Audio["Audio Server (AI/Python)"]
        TTS -->|Events| Mix["Soundscape Mixer"]
    end

    subgraph "Data & State"
        NATS["(NATS JetStream)"]
        PG["(PostgreSQL)"]
        Users["User Database Service"]
    end

    UI --> NATS
    Estimate --> NATS
    PDF --> NATS
    OCR --> NATS
    TTS --> NATS
    Mix --> NATS
    Users --> PG
```

### Core Services

1.  **UI Service (BFF):** A robust Go backend serving a modern React/Vite frontend. Handles authentication, file uploads, and real-time status updates via the Bridge Service.
2.  **User Database Service:** The central source of truth for identity, authentication (Argon2id/JWT), and job state management, backed by PostgreSQL.
3.  **Cost Estimator:** A specialized worker that analyzes document density (character count) to provide precise, real-time pricing quotes.
4.  **PDF-to-PNG Service:** Renders PDF pages into high-quality images and uses Gemini to generate document-wide narration and music directives.
5.  **PNG-to-Text Service:** Uses Gemini-powered OCR to extract and normalize text, optimized for spoken narration (STEM expansion, code block handling).
6.  **Summarization Service:** An optional pre-processor that uses GenAI to condense lengthy documents into concise summaries before they enter the main pipeline.
7.  **TTS Service:** Orchestrates speech synthesis and music generation, managing workload distribution across GPU-accelerated workers.
8.  **Audio Server (Python):** A dedicated AI inference engine hosting state-of-the-art Voice Cloning models (VoxCPM 1.5).
9.  **Soundscape Service:** Generates tailored background music and ambient soundscapes using Google Lyria.
10. **Audio Mixer Service:** Aggregates speech chunks, truncates silence, and mixes in background music to produce the final audio artifact.
11. **Bridge Service:** Synchronizes asynchronous backend events with the User Database Service to provide real-time status to the UI.
12. **NATS Orchestrator Service:** Provisions and manages the NATS JetStream infrastructure (Streams, Buckets, KV).

### Foundation & Tooling

*   **Common Worker:** A generic Go library providing standardized NATS JetStream consumption, heartbeats, and infrastructure synchronization patterns.
*   **Common Events:** The central source of truth for all asynchronous communication, defining canonical NATS subjects and strictly typed event schemas.
*   **Zig Common HTTP:** A high-performance, non-blocking HTTP server library in Zig, used as the networking backbone for Go and Python services requiring maximum throughput.
*   **C++ Common Client:** A provider-agnostic C++ library for LLM APIs (Gemini, OpenAI), exposed via a C ABI for seamless integration with Go and Python.
*   **Tool Executor:** A high-integrity Zig/Python service providing atomic file operations and agent coordination via the Model Context Protocol (MCP).
*   **Helper Scripts:** A suite of orchestration tools for local environment setup, service management, and end-to-end testing.

---

## ⚡ Technology Stack

*   **Communication:** NATS JetStream (Streams, KV, Object Store) for ultra-low latency, persistent messaging.
*   **Backend:** Go (Golang) 1.24+ for high-concurrency microservices.
*   **Tooling:** Zig 0.15.2 for high-integrity systems programming.
*   **AI/ML:** Python 3.12+ with PyTorch and CUDA for neural inference.
*   **Frontend:** React 18, TypeScript, Vite, Material UI (MUI) for a responsive, dark-mode "Studio" experience.
*   **Database:** PostgreSQL with `pgx` and automated migrations.

## 🔄 The Event-Driven Workflow

1.  **Ingestion:** User uploads a PDF. The **UI Service** streams it to the **NATS Object Store** (`PDF_FILES` bucket).
2.  **Estimation:** The **Cost Estimator** fetches the file, calculates character density, and returns a binding quote.
3.  **Summarization (Optional):** If requested, the **Summarization Service** generates a condensed version of the document and re-injects it into the pipeline as a new PDF.
4.  **Payment & Trigger:** Upon user approval, a `pdfs.created` event is published to the `PDFS` stream.
5.  **Decomposition:** The **PDF Service** explodes the document into high-res images (`PNG_FILES` bucket) to capture visual context.
6.  **Extraction:** The **PNG-to-Text Service** extracts text (`TEXT_FILES` bucket), filtering out artifacts (headers/footers).
7.  **Performance:** The **TTS Service** requests audio generation from the **Audio Server**, chunk by chunk.
8.  **Production:** The **Audio Mixer Service** aggregates the audio chunks, overlays the selected musical theme, and renders the final `.wav` artifact in the `AUDIO_FILES` bucket.

## 🛡️ Operational Excellence & Philosophy

This project adheres to the **Manifesto of Truth**:
- **Reality is in the Files:** We prioritize current code and logs over internal assumptions.
- **Whole Words Only:** Variable and function naming must be explicit and clear (e.g., `context`, not `ctx`).
- **Love & Craftsmanship:** We write code for the human who has to read it at 3 AM.
- **High Integrity:** All changes are atomic, verified, and strictly documented.

---

*Built with ❤️, Craftsmanship, and Discipline.*
