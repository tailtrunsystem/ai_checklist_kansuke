---
description: Auditor for Kansuke Multi-source (App, Photo, Kensa)
alwaysApply: true
---

# Kansuke Project Rules

- **Cross-Source Awareness**: You must always consider the dependencies between **Kansuke_app**, **Kansuke_photo**, and **Kansuke_kensa**.
- **Japanese Naming Conventions (Strict)**:
  - Photo app: **工程写真アプリ**
  - Kensa app: **検査点検アプリ**
  - Main/Launch app: **かん助Launch**
- **Audit Logic (PVAH Matrix)**: 
  - IF a change occurs in any folder within `kansuke-app/packages/` or any file imported via `package:common/common.dart`, it is **MANDATORY** to perform a cross-resource search (`grep`) in all 3 modules (`kansuke-app`, `kansuke-photo`, `kansuke-kensa`).
  - You MUST generate an explicit **Impact Matrix** in the checklist for every CR.
- **Formatting**: Strictly follow the Expert MD template (Header metadata, Deep Audit, 7-column table).

## QC Logic & Test Scenarios

### 1. Photo App Granularity (Toritame vs Kotei)
- **Toritame (撮りため)**: Fast capture (no project search). Test: Offline, background restore, sync queue visibility.
- **Kotei (工程写真)**: Normal project-linked. Test: Folder focus, QR mapping.
- **Conflict**: Test switching between modes in one session.

### 2. Comprehensive Upload Flow Requirements
- **Kensa (検査点検アプリ)**: 
  - **Upload Trial**: Sync trial data. MUST verify project selection dialog appears correctly.
  - **SLL**: 50+ inspections. Monitor progress bar stability.
  - **Multi-device**: Concurrent sync. Verify no data duplication.
- **Photo (工程写真アプリ)**:
  - **QR Code Flow**: Online mapping vs Offline metadata retention.
  - **SLL**: 100+ photos. Background upload survival.
  - **Interruption**: Resume from last successful item on network restoration.
- **Main (Launch)**:
  - **Sync Visualization**: Cloud icon badge mirrors total combined queue.
  - **Batch Upload**: Unified trigger for all pending items.

### 3. Login & Authentication Logic
- **Session Propagation**: Activity in one app activates all other apps.
- **Login Re-auth**: Apps must auto-activate on focus if another app refreshed the token.

### 4. Account & Dealer Security (FLUTTER-165)
- **Critical Risk**: Unsent data leaking between accounts.
- **Verification**: Login Dealer A (create unsent) -> Logout -> Login Dealer B.
  - **Launch Header**: MUST be disabled for Dealer B.
  - **Sync List**: MUST NOT show Dealer A's files.
  - **Relogin A**: Icon MUST re-activate immediately.

### 5. No-Omission Verification Rule (Crucial)
- **Mandatory Logic**:
  1. IF a reference file (CR or MD) is provided, you MUST perform a line-by-line sync.
  2. EVERY scenario in the reference MUST be included in the new generation.
  3. NEVER discard old valid test cases for new ones; they must coexist.