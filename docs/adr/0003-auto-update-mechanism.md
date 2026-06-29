# ADR 0003: Auto-Update Mechanism and Distribution Strategy

**Date:** 2026-06-29

## Status
Accepted

## Context
Arc requires a self-hosted update mechanism via GitHub Releases, bypassing traditional app stores. The update experience must be non-intrusive and seamless, avoiding modal dialogs that interrupt the user's workflow. Furthermore, the distribution format on Windows needs to align with Arc's local-first, privacy-focused philosophy.

## Decision
1. **Update Strategy:**
   - **Silent Background Download:** The app will poll `api.github.com` for the latest release on startup. If a new version exists, it will download the full package in the background.
   - **Non-intrusive Prompt:** Once downloaded, a glowing glassmorphic button will appear in the UI overlay, prompting the user to "Restart to Update."
   - **Resilience:** GitHub API 403 errors (rate limits) and interrupted downloads will fail silently and retry on the next startup. Failed updates will trigger an automatic rollback.
2. **Windows Distribution:**
   - Arc will be distributed as a **Portable ZIP** rather than an installer (e.g., MSIX/NSIS).
   - Updates will be handled by a dedicated, minimal `updater.exe` script. When the user clicks the update button, Arc will extract the new ZIP, launch `updater.exe`, terminate itself, allow the updater to swap the files, and then be relaunched by the updater.
3. **Android Distribution:**
   - Arc will download the new APK to the cache directory.
   - Clicking the update button will invoke the system Package Installer via Intent.

## Consequences
- **Positive:** Delivers a modern, browser-like update experience (similar to Chrome/VSCode). The Portable ZIP format reinforces the "your data is yours" philosophy since the entire app and data can be easily moved on a USB drive.
- **Negative:** Managing a separate `updater.exe` lifecycle on Windows introduces OS-level edge cases (file locking, permissions) that must be handled robustly. Full package downloads use more bandwidth than patch updates.