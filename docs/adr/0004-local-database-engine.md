# ADR 0004: Local Database Engine

**Date:** 2026-06-29

## Status
Accepted

## Context
Arc requires a robust, high-performance local database to manage thousands of Nodes and tens of thousands of Photos, including the complex many-to-many relationships between them. Fast querying by temporal ranges (Chunks) is critical for rendering the Flame 3D Starfield without dropping frames.

## Decision
We will use **Isar** as the primary local database engine.

1. **Engine:** Isar provides a type-safe Dart API, excellent performance (built on SQLite), and native support for many-to-many relationships (`IsarLinks`).
2. **Schema:**
   - `Nodes`: id, date, title, description, type, latitude, longitude, createdAt
   - `Photos`: id, filePath, thumbPath, takenAt, latitude, longitude, dominantColor, hash, width, height
   - Relationships managed via `IsarLinks` (no manual join tables).
3. **Query Optimization:** Indexes will be strictly applied to `date` (for Chunk queries), `hash` (for deduplication), and `takenAt`.
4. **Fallback:** If Isar presents critical compatibility issues on Windows desktop during Phase 1, the fallback is **Drift**.

## Consequences
- **Positive:** Greatly simplifies relationship management and schema migrations. Provides the necessary performance for 60fps Starfield viewport queries. Easy to bundle into `.arc` archive files for cross-device sync.
- **Negative:** Isar relies on native binaries, which may occasionally cause build issues across different platforms (Windows vs Android) compared to pure Dart solutions, necessitating the Drift fallback plan.