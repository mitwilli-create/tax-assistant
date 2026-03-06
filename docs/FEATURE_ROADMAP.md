Tax Assistant — Feature Roadmap
Strategic Development Plan  |  Version 1.0  |  March 2026
Prepared by: Mitchell Williams  |  Project: AI Tax Assistant (Claude Project)
Executive Summary
The Tax Assistant is an AI-powered, multi-session tax preparation system built on Claude Projects, purpose-built to manage the complexity of multi-state filing, RSU income allocation, and year-over-year continuity. This roadmap outlines the planned development trajectory across three six-month phases — from a stable v1.0 baseline through a fully autonomous, compliance-aware v3.0 platform.

Phase 1: Foundation & Accuracy (Q2–Q3 2026)
Focus: Solidify the core filing workflow, eliminate manual data re-entry, and establish a verified accuracy baseline.

Phase
Features & Enhancements
1.1
Q2 2026
Automated 1099-B ingestion: parse Schwab CSV/PDF directly into pre-filled Form 8949 entries
RSU lot-by-lot reconciliation tool: match W-2 Box 1 income to vesting events with cost-basis verification
Multi-state wage allocation calculator: formalize the NY/IL back-calculation methodology into a reusable module
Document completeness checker: flag missing or mismatched tax documents before filing session begins
1.2
Q3 2026
Year-over-year data carryforward: automatically import prior-year AGI, carryforward losses, and state credits
FreeTaxUSA alert interpreter: classify software alerts by risk level (informational / warning / critical)
Withholding optimizer: model W-4 scenarios to hit target refund within $500
Audit risk dashboard: score each line item by IRS audit probability and flag high-risk entries

Phase 2: Intelligence & Automation (Q4 2026–Q1 2027)
Focus: Shift from reactive Q&A to proactive planning — with AI-driven deduction discovery and scenario modeling.

Phase
Features & Enhancements
2.1
Q4 2026
Deduction discovery interview: structured, adaptive questionnaire that surfaces overlooked deductions (home office, professional development, medical, charitable)
Covered call tax treatment engine: track option premium income, expired contracts, and assigned positions for Schedule D reporting
Qualified dividend optimizer: validate broker QDIV classification and model impact of holding period changes
IRS transcript reader: parse IRS account transcripts to auto-detect processing status, credits applied, and balance due
2.2
Q1 2027
Multi-year tax projection model: forecast 2027-2030 liability under multiple income scenarios (RSU cliff vesting, covered call income, salary increases)
State nexus analyzer: assess whether remote work creates new state filing obligations
NIIT exposure calculator: track net investment income tax threshold in real time as investment income accumulates
Integrated knowledge base versioning: diff-aware updates to project files across sessions without manual re-upload

Phase 3: Compliance Platform (Q2–Q3 2027)
Focus: Expand from a personal assistant to a compliance-grade platform with audit readiness, regulatory monitoring, and structured output generation.

Phase
Features & Enhancements
3.1
Q2 2027
Audit defense dossier generator: compile supporting documentation for each Schedule D entry, RSU lot, and deduction claim
IRS regulation change monitor: weekly digest of new Revenue Procedures, Notices, and final regulations affecting the filing profile
Form pre-fill export: generate a structured JSON payload compatible with direct API submission to tax software platforms
Amended return workflow: detect prior-year errors and generate Form 1040-X entries with explanation narratives
3.2
Q3 2027
Multi-user mode: support for household or advisor-shared access with role-based document permissions
CPA handoff package: auto-generate a structured summary document with all computed figures, assumptions, and supporting docs for professional review
Estimated payment scheduler: calculate quarterly safe-harbor payments and generate IRS Direct Pay reminders
Natural language regulatory citations: every recommendation automatically linked to the specific IRC section, IRS publication, and form instruction

Feature Priority Matrix
The following matrix summarizes all planned features by priority, implementation effort, and current status.

Feature
Priority
Effort
Status
1099-B automated ingestion
High
Medium
Planned — Q2 2026
RSU lot reconciliation tool
High
High
Planned — Q2 2026
Multi-state wage allocator
High
Medium
Planned — Q2 2026
FreeTaxUSA alert interpreter
High
Low
Planned — Q3 2026
Withholding optimizer (W-4)
High
Medium
Planned — Q3 2026
Deduction discovery interview
Medium
Medium
Planned — Q4 2026
Covered call tax engine
Medium
High
Planned — Q4 2026
Multi-year projection model
Medium
High
Planned — Q1 2027
Audit defense dossier
Medium
High
Planned — Q2 2027
CPA handoff package
Low
Medium
Planned — Q3 2027
Estimated payment scheduler
Low
Low
Planned — Q3 2027

Success Metrics
Accuracy
Zero material errors in filed returns across all jurisdictions
100% of IRS alerts resolved with documented rationale before submission
RSU cost basis variance < $1 per lot after reconciliation

Efficiency
Filing session time reduced from 4+ hours to under 30 minutes by Phase 2
Document preparation completeness rate of 100% before session start
Zero manual re-entry of figures that appear in uploaded source documents

User Experience
Every recommendation includes a cited IRS publication or IRC section
All state allocation figures independently verifiable from primary documents
Session state fully restorable from knowledge base after any gap of 30+ days

Tax Assistant Feature Roadmap  |  Confidential  |  Mitchell Williams  |  March 2026