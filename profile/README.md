# Book Expert Project

This repository contains a collection of services and tools designed to process PDF documents into audio books using an event-driven architecture based on NATS and JetStream.

## Project Structure

- `configurator/` - Configuration management tools
- `docs/` - Documentation files
- `events/` - Event definitions and handling
- `project-toml/` - Project configuration in TOML format
- `wav-aggregator-service/` - Service for combining WAV files

## Architecture Overview

The system uses a resilient, event-driven microservices architecture built on NATS and JetStream. Services are decoupled and communicate asynchronously by passing messages containing references to data rather than the data itself.

## NATS Pipeline Flow

```mermaid
graph TD
    A[PDF Upload] -->|pdf.created| B(pdf-to-png-service)
    B -->|png.created| C(png-to-text-service)
    C -->|text.processed| D[tts-service]
    D -->|audio.chunk.created| E[pcm-to-wav-service)
    E -->|wav.created| F[wav-aggregator-service]
    F -->|final.audio.created| G[Final Audio Files]
    
    style A fill:#e1f5fe
    style B fill:#f3e5f5
    style C fill:#e8f5e8
    style D fill:#fff3e0
    style E fill:#fce4ec
    style F fill:#f1f8e9
    style G fill:#e0f2f1
    
    linkStyle 0 stroke:#1976d2
    linkStyle 1 stroke:#7b1fa2
    linkStyle 2 stroke:#388e3c
    linkStyle 3 stroke:#f57c00
    linkStyle 4 stroke:#c2185b
    linkStyle 5 stroke:#689f38
```

## Core Concepts

### NATS & JetStream

NATS provides the core messaging backbone. JetStream is used for all inter-service communication, guaranteeing that messages will be retained and processed even if a consuming service is offline (at-least-once delivery).

### JetStream Object Store

To handle large binary files (PDFs, PNGs, audio), the JetStream Object Store is used. Instead of publishing large payloads directly into a NATS message, services upload files to designated Object Store buckets and publish lightweight event messages containing keys to reference those files.

## Services

1. `pdf-to-png-service` - Converts PDF pages to PNG images
2. `png-to-text-service` - Performs OCR on PNG images to extract text
3. `tts-service` - Generates raw PCM audio from text
4. `pcm-to-wav-service` - Converts PCM audio to WAV format
5. `wav-aggregator-service` - Combines multiple WAV files into a single audio book

## Configuration

The central `project.toml` file defines all NATS-related configuration including stream names, subject names, and object store bucket names.