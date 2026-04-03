---
description: Autonomous Expert QA Checklist Generator for Kansuke Projects
---

[CONTEXT]
Reference @.agents/CONTEXT.md for architecture and @template_checklist_task_no_xx.numbers for template structure.

[INPUT]
Task ID: {{task_id}} (e.g., CANSUKE_FLUTTER-181)
CR: {{cr}} (Reference document or description)
PVAH (Optional): {{pvah}}

[STRICT ARCHITECTURE AUDIT]
1. **PVAH Audit Matrix**: Mandatorily check cross-app impact (Launch, Kensa, Photo).
2. **Deep Audit Analysis**: Identify logic risks: DB (Kotei/Toritame), SLL Stability, Lifecycle, Connection, and **Account/Dealer Security (FLUTTER-165)**.

[OUTPUT 1: EXPERT MD FILE (VIETNAMESE)]
Create `{{task_id}}/KANSUKE_Checklist_{{task_id}}.md` with:
- Header metadata (Creator, Date, OS, Total Items).
- Overview of 3 apps and their specific roles.
- PVAH Matrix.
- **Deep Audit Table** (Linking logic risks to test categories).
- **7-column Checklist Table** (Separate Procedure / Expected Result / App Name / Environment).

[OUTPUT 2: DECOUPLED DATA (JSON)]
- Create `{{task_id}}/data.json` (Japanese test cases).
- Create `{{task_id}}/history.json` (Japanese logs).
- **Rule**: NO hardcoding in gen_xlsx.py. All data must reside here.

[OUTPUT 3: EXCEL GENERATION]
Run `python3 gen_xlsx.py {{task_id}} {{task_id}}`.

[STRICT VERIFICATION CHECK (MANDATORY BEFORE DELIVERY)]
Confirm the following to the user:
1. **No-Omission**: "I have cross-checked the reference (CR) line-by-line and confirmed 0 scenarios were omitted."
2. **Strict Naming**: "I have used strictly [工程写真アプリ], [検査点検アプリ], [かん助Launch] in all columns."
3. **History Sync**: "I have appended a substantive log entry for this generation/update in history.json."
4. **Account Security**: "I have verified Dealer/Account switch scenarios are included for session-related tasks."

[CORE RULES]
1. **Layout Freeze**: Do not touch the Excel formatting logic in Python unless explicitly told.
2. **Japanese vs Vietnamese**: MD is Vietnamese; XLSX/JSON data is strictly Japanese.
3. No conversational filler. Deliver files and wait for confirmation.