Technical Specification (Improved): Advanced Medical Device Application, Attachment Form & Agentic Review System (Streamlit + HF Spaces)
Scope: This specification upgrades the existing Hugging Face Spaces Streamlit system while preserving all original features (Dashboard, TW Premarket, 510(k) Intelligence, PDF→Markdown, 510(k) Review Pipeline, Note Keeper & Magics, Agents Config, multi-provider LLM routing, painter styles, bilingual UI, status indicators, etc.). It adds a new application/attachment form workflow based on the provided TFDA “許可證有效期間展延” checklist-like PDF content, plus additional “WOW UI” requirements and enhanced agent chaining controls—without prescribing code changes.

1. Goals & Product Principles
1.1 Primary Goals
Regulatory-grade form handling: Provide a structured, auditable way to capture and review TFDA license extension (有效期間展延) submission requirements, including applicability, uploaded artifacts, and review results.
Human-in-the-loop control: Users can override prompts, models, and max-tokens before running agents step-by-step, and can edit an agent’s output as input to the next agent.
WOW UX: Deliver a polished interface with theme controls (Light/Dark), bilingual (English/繁體中文), painter-style aesthetics (20 styles with Jackpot), status indicators, and an interactive dashboard.
Secure API key handling: Allow API keys entered in the UI only if not present in environment; do not reveal environment keys.
1.2 Non-Goals
No direct e-submission to TFDA/FDA portals.
No guarantee of regulatory approval; system provides structured assistance and gap analysis.
2. System Overview & Architecture
2.1 Deployment Context
Platform: Hugging Face Spaces
App Framework: Streamlit
Configuration: agents.yaml for agent definitions; optional SKILL.md knowledge/instructions
LLM Providers: OpenAI, Google Gemini, Anthropic, Grok (xAI) via API keys
2.2 High-Level Components
UI Shell (WOW UI Layer) Theme, language, painter style, dashboard, and global navigation.
Artifact Workspace Persistent in-session objects: Application drafts, guidance markdown, review reports, notes, attachment registry.
LLM Gateway Routes calls to provider based on model ID; enforces max_tokens defaults; centralizes redaction & logging policy.
Agent Orchestrator (Step Runner) Runs agents sequentially; allows prompt/model/token override per step; supports “edited output → next input”.
Document Processing PDF page range extraction, OCR option (vision-capable models), markdown normalization, keyword highlighting.
Config Manager View/edit/upload/download agents.yaml and SKILL.md; optional standardization agent.
3. WOW UI / UX Requirements
3.1 Global UI Controls (must exist)
Theme Toggle: Light / Dark.
Language Toggle: English / 繁體中文; all major UI labels localized.
Painter Style Selector: 20 painter-inspired styles; includes Jackpot random picker.
Typography:
UI: Inter/Roboto equivalent (Streamlit default acceptable with CSS enhancements).
Editors: Monospace (JetBrains Mono style if available).
3.2 WOW Status Indicators (system-wide)
Each major module must show:

Step Status: pending / running / done / error (with colored dot).
Completeness/Readiness meters where applicable:
TFDA Premarket Application completeness (existing).
New TFDA License Extension Attachments completeness (added).
3.3 Awesome Interactive Dashboard (enhanced)
Dashboard must include:

Run Metrics: total runs, unique tabs, estimated token consumption.
Latest Activity Card: last run tab/agent/model/tokens/time, with gradient severity based on token volume.
Charts:
Runs by tab (bar)
Runs by model (bar)
Tab×Model heatmap
Token usage over time (line)
New: “Operational Health” panel:
API key availability (provider readiness icons)
Last error summary (if any)
Current theme/language/style snapshot
4. API Key Management Requirements (Security + UX)
4.1 Priority Rules
Environment variables are primary (HF Secrets).
If missing, allow user to input keys in sidebar.
4.2 Display Rules
If key exists in environment: show a neutral message like “Key loaded from environment.” Must not reveal the key or length.
If user inputs key: mask input; store in memory for session only.
Never log keys to console, charts, or history.
4.3 Provider Coverage
Keys supported:

OPENAI_API_KEY
GEMINI_API_KEY
ANTHROPIC_API_KEY
GROK_API_KEY
5. Model & Agent Execution Controls (Step-by-Step Chaining)
5.1 Supported Model List (UI selectable)
gpt-4o-mini, gpt-4.1-mini
gemini-2.5-flash, gemini-3-flash-preview, gemini-2.5-flash-lite, gemini-3-pro-preview
Anthropic models (as configured in deployment; UI must list configured IDs)
grok-4-fast-reasoning, grok-3-mini
5.2 Per-Agent Overrides (must be editable before run)
For each agent execution block:

Editable Prompt (pre-filled by agent template)
Editable Model selector
Editable max_tokens (default 12000)
Optional temperature control (global default; per-agent optional)
5.3 Editable Outputs as Next-Step Inputs
Each agent output must be shown in:
Markdown view (editable)
Plain text view (editable)
The user can choose which edited output becomes the next agent’s input (“Use as Next Input” behavior).
Maintain provenance:
Record: agent id/name, selected model, prompt snapshot hash/time, token estimate.
6. New Feature: TFDA “許可證有效期間展延” Application & Attachment Form
This feature is derived from the user-provided checklist content containing sections 一～九 and their applicability states.

6.1 User Story
“As a TFDA applicant or reviewer, I want to capture whether each required attachment is applicable, upload/track files and metadata (e.g., copy/original, issue date, reference case number), run an agentic completeness/relevance check against guidance, and generate a structured Markdown report.”

6.2 New Module Placement
Add a new workflow inside TW Premarket as either:

A dedicated sub-tab: 「許可證展延」/ “License Extension”, or
A new top-level tab: TFDA License Extension (展延)
Must be consistent with bilingual controls.

6.3 Form Sections & Data Capture (I–IX)
The system must represent each section as a structured entity with:

Section ID (1–9)
Section Name (Traditional Chinese + English label)
Applicability: 適用 / 不適用 / 不明
Vendor self-evaluation text (if present)
Uploaded files registry (0..n)
Review checklist items (where applicable)
Status: missing / partially complete / complete
6.3.1 Sections (as provided)
一、醫療器材許可證有效期間展延申請書 (Applicable)
二、原許可證 (Applicable)
標籤、中文核定說明書或包裝核定本 (Applicable; note: update system labeling file if complete)
三、出產國製售證明 (Applicable; includes copy/original, original stored at ref no, issue date)
四、原廠授權登記書 (Applicable; includes original stored at ref no, issue date)
五、QMS/QSD 證明文件 (Applicable; includes original/copy, issue date)
六、第一等級查驗登記申請書 (Not applicable)
七、製造業/販賣業醫療器材商許可執照 (Applicable)
八、委託製造核准證明 / 委託製造契約 / 委託製造契約(委託者為國外業者) (Not applicable in example; includes a built-in review checklist when applicable)
九、安全監視或上市後研究計畫報告 (Not applicable)
Note: Although numbered “九” at end, section 3 (“標籤…”) appears between 二 and 三 in the provided content. The UI must support “display order” separate from “logical numbering” and keep the exact original labels.

6.4 File Metadata Schema (per uploaded artifact)
For each uploaded file:

file_name (original)
description (user-entered)
version_tag (optional)
doc_type (copy/original/unspecified)
issue_date (optional; ROC date input supported, stored normalized to ISO)
reference_case_no (e.g., “1110815922”, “1120719945 變更案”)
status_flag: active / voided / canceled (對應 作廢/註銷)
notes (free text)
uploaded_at timestamp (UTC)
sha256 hash (optional; for audit, in-memory or persisted if allowed)
6.5 Built-in Checklist for Section 八 (委託製造) when Applicable
If Section 八 is marked “適用”, present checklist rows:

委託者與受託製造業者名稱/地址與原許可證一致
委託製程及分級分類品項與原許可證資料相符
契約於有效期限內（for contract variants)
Each row must support:

Result: 符合 / 不符合 / 不適用
Reviewer notes
Evidence link (select an uploaded file)
6.6 Completeness Scoring for License Extension Packet
Compute a score (0–100%) using:

Applicable sections must have at least one file OR a justified explanation.
For sections requiring metadata (e.g., issue date), missing metadata reduces completeness.
If a section is “不適用”, it does not count against completeness.
The UI must display:

Completeness percentage card (WOW gradient)
A progress bar
A “Missing Items” list with direct anchors to sections
6.7 Generated Outputs (Artifacts)
License Extension Packet Summary (Markdown)
Section-by-section table: Applicability, files count, metadata completeness, notes.
Gap Analysis Report (Markdown) (agent-produced)
Missing/unclear items
Consistency checks (e.g., original license vs manufacturer info)
Recommended actions
Export:
JSON export of structured packet registry
CSV export (flattened)
Markdown export of summary + report
7. Guidance Processing (PDF/Text/MD) – retained & strengthened
7.1 Page Trimming
Users specify page ranges (e.g., 1-5, 10-12) for guidance.

7.2 OCR Modes
Native extraction for text PDFs
LLM Vision OCR toggle for scanned/complex layouts (OpenAI/Gemini vision-capable models)
7.3 Coral Keyword Highlighting (retained)
Regulatory keywords wrapped with coral color spans:

MUST/SHALL/REQUIRED
510(k), Predicate, TFDA, QMS, labeling, sterilization, etc.
8. Agent Catalog (agents.yaml) – New/Updated Agents
8.1 Required Agents (additive, keep originals)
tw_license_extension_packet_builder Input: form fields + file registry dump Output: structured Markdown summary + completeness commentary
tw_license_extension_gap_review_agent Input: packet summary + guidance text Output: gap analysis report + action list + risk notes
tw_attachment_consistency_agent Input: packet + TW application info (if available) Output: mismatch detection (names, addresses, dates, license references)
pdf_to_markdown_agent (existing)
tw_screen_review_agent (existing)
tw_app_doc_helper (existing)
8.2 Agent Standard Schema (enforced)
Each agent entry must include:

name, description
model, temperature, max_tokens
system_prompt
user_prompt_template (optional)
9. AI Note Keeper (retained) + Expanded “AI Magics” (6 features)
9.1 Base Flow (retained)
Paste notes → transform into organized Markdown
Keywords highlighted in coral
Editable in Markdown or text view
Prompt and model selectable
9.2 Six “AI Magics” (must exist)
AI Formatting: normalize headings/lists without changing meaning
AI Keywords: user provides keywords + color picker; highlights keywords in rendered note (HTML span)
AI Summary: bullets + short executive summary
AI Action Items: extracts tasks into a table (priority, owner placeholder)
AI Glossary: term table (Term, Chinese/full name, explanation)
AI Risk Flags (new): detects regulatory risk signals in the note (e.g., missing validation, unclear intended use) and outputs:
Risk category
Trigger text snippet
Suggested mitigation question
10. Navigation & Workflows (End-to-End)
10.1 TW Premarket (existing) + License Extension (new)
Fill TW application → generate Markdown
Load guidance (optional)
Run screen review agent → edit output
Run doc helper agent → edit output
License Extension:
Mark applicability + upload artifacts + fill metadata
Generate packet summary (agent or deterministic)
Run gap analysis agent against guidance
Export bundle artifacts
10.2 510(k) Pipelines (existing)
Keep current “submission structuring → checklist → review report → editable outputs”.
11. Data Handling, Privacy, and Auditability
11.1 Session Storage
Use Streamlit session state for:
Form fields
Uploaded file metadata registry
Agent prompts, model overrides, outputs, edited outputs
Run history (without secrets)
11.2 Logging Policy
Store only:
Timestamps, tab, agent name, model id, token estimate, status
Never store API keys, raw environment variables, or hidden secrets.
11.3 Export Controls
Exports must be explicit user actions (download buttons).
When exporting JSON/CSV, include a disclaimer header block (non-regulatory advice).
12. Acceptance Criteria (Testable)
User can switch Light/Dark, English/繁體中文, and painter styles; Jackpot selects randomly among 20 styles.
If environment keys exist, UI indicates presence without revealing key; if not, allows password input.
For any agent step, user can edit prompt/model/max_tokens (default 12000) before running.
User can edit an agent’s output in markdown/text and pass edited output to next agent.
License Extension module supports sections I–IX with applicability, file registry, metadata, and completeness scoring.
Section 八 checklist appears only when applicable and records results and evidence.
System generates downloadable Markdown summary and JSON/CSV exports for the license extension packet.
Dashboard shows run analytics and provider readiness indicators.
13. 20 Comprehensive Follow-up Questions
For the 許可證展延 workflow, do you want the system to support multiple products/licenses per session, or strictly one license packet at a time?
Should the license extension module store both ROC dates and Gregorian dates in the UI, and what is the authoritative export format?
For “作廢/註銷” states, do you need a full change history (who/when) or only current status per file?
Do you want to enforce file type restrictions per attachment section (PDF only vs PDF/DOCX/IMG)?
Should the system perform automatic extraction of issue dates/reference numbers from uploaded PDFs/images via OCR, or keep it manual-first?
What is the required level of validation for key fields (e.g., uniform ID format, email, phone), and should invalid fields block report generation?
For Section 三/四/五 metadata (“正本在…”, “出具日期…”), do you want standardized fields for issuing authority and certificate number as well?
For Section 八 (委託製造), do you need support for multiple subcontractors and multiple contracts, each with its own checklist?
Should the TFDA module integrate with the existing TW premarket application form to auto-fill shared fields (manufacturer name/address), and which fields are authoritative if conflicts arise?
Do you require a document naming convention generator (e.g., “03_CFS_YYYYMMDD.pdf”) to standardize uploads?
For the agent chain, should there be a visual pipeline editor (drag/drop steps) or is linear step-by-step sufficient?
Do you want per-agent temperature controls exposed in the UI, or only a global temperature?
Should the system support saving/loading sessions (persisting to disk or HF storage), or remain session-only for privacy?
For bilingual UI, should generated agent outputs default to the UI language, or be controlled per-agent?
Which Anthropic model IDs should be included in the selectable list (you mentioned “anthropic models” broadly)—do you have a fixed set approved?
Do you want a provider fallback strategy (e.g., if Gemini fails, retry with OpenAI) or strict user-selected provider only?
For keyword highlighting, do you want a custom keyword dictionary per project (uploadable), and should it apply to guidance, notes, and reports uniformly?
Should the dashboard include cost estimation by provider/model (rough USD estimate), or only token estimates?
For compliance reports, do you need formalized scoring (Pass/Fail/Clarification) mapped to TFDA criteria similar to the FDA compliance matrix?
What is the expected maximum PDF size/page count and concurrent users per Space, so we can define performance limits and recommended HF hardware tiers?
