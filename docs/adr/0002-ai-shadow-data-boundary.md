# ADR 0002: AI Data Boundary and Shadow Data Architecture

**Date:** 2026-06-29

## Status
Accepted

## Context
Arc plans to integrate AI capabilities (LLM/VLM) in future versions to automatically generate node descriptions and tags from photos. However, the app's core philosophy is local-first and user-driven. We must ensure that unpredictable AI-generated content never silently overwrites user-curated data, and we must manage the performance cost of local algorithms (like K-Means color extraction) without blocking the UI.

## Decision
1. **Shadow Data Architecture:**
   - All AI-generated content will be stored in dedicated "shadow" fields (e.g., `ai_description`, `ai_suggested_tags`) in the database.
   - AI content will never overwrite the primary user fields (`description`, `tags`) automatically.
   - The UI will present shadow data with a distinct visual marker (the "✦" symbol) and require explicit user action ("Adopt") to promote the data to the primary fields.
2. **Privacy & API Configuration:**
   - AI configurations (Provider, Base URL, API Key, Model) will be stored locally.
   - When AI features are activated, a strict privacy opt-in is required before any image is sent to a third-party API.
   - Images sent to APIs will be aggressively downscaled (max 512x512).
3. **Local Processing (K-Means):**
   - We reject the use of C++/Rust FFI for image processing to keep the build process simple.
   - Instead, computationally expensive algorithms like K-Means color extraction will run in a Dart `Isolate` background queue.
   - These algorithms will operate exclusively on heavily downscaled thumbnails (e.g., 50x50), ensuring the main UI thread remains completely fluid.

## Consequences
- **Positive:** Protects user data integrity perfectly. Ensures the app remains responsive during mass photo imports. Prepares the database schema for Phase 4+ without polluting Phase 1-3 logic.
- **Negative:** Slightly increases database schema complexity by duplicating fields. Requires careful state management to show both user data and AI suggestions simultaneously in the UI.