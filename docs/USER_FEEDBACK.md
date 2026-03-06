Tax Assistant — User Feedback Compilation
Beta Testing Summary  |  Version 1.0  |  March 2026
Primary Tester: Mitchell Williams  |  Tax Year: 2025  |  Platform: Claude Projects
Testing Overview
This document compiles feedback gathered during beta testing of the Tax Assistant across the 2025 tax filing season (October 2025 through March 2026). Testing spanned multiple multi-session workflows including document ingestion, multi-state wage allocation, RSU cost basis reconciliation, and FreeTaxUSA platform integration. Feedback is synthesized from session logs, in-session correction events, and post-session reflection notes.

Sessions Logged
Documents Processed
Errors Caught by User
States Filed
12+
18
4
3

Satisfaction Ratings
Ratings assessed by the primary tester across key functional dimensions after the 2025 filing cycle.

Dimension
Rating
Score
Key Observation
Document Accuracy
★★★★★
4.5/5
Excellent extraction from W-2, 1099-DIV, 1099-R PDFs
Multi-State Complexity
★★★★☆
4/5
NY wage back-calculation methodology was sound
Proactive Error Flagging
★★★★★
4.5/5
Caught NY overpay scenario before it was filed
Instruction Specificity
★★★★☆
4/5
Field-level guidance was clear and actionable
Uncertainty Communication
★★★★★
5/5
Consistently flagged inferred vs. document-sourced figures
Citation Reliability
★★★★☆
3.5/5
IRS publication references were accurate but inconsistently applied
Session Continuity
★★★☆☆
3/5
Context loss between sessions required re-uploading documents
1099-B Handling
★★★☆☆
2.5/5
Manual entry required; no automated ingestion yet
Overgeneralization Risk
★★★★☆
3.5/5
AGI field instruction issue caught and corrected mid-session

Verbatim Feedback Highlights
Positive Feedback — What Worked Well


“The assistant caught that FreeTaxUSA pre-filled $335,340 for NY wages when the correct figure was ~$10,000. That single catch would have cost me roughly $19,000 in excess tax liability. That's not a nice-to-have — that's the whole point.”
— Mitchell Williams, Primary Tester    [Critical Error Prevention]


“I appreciated that whenever a number was derived by inference rather than pulled directly from a document, the assistant said so explicitly. That's exactly the behavior I want — it makes me verify rather than trust blindly.”
— Mitchell Williams, Primary Tester    [Uncertainty Transparency]


“The pre-computation approach was brilliant for my situation. I was recovering from surgery and needed to file with minimal cognitive load. Having every field value computed and ready to copy-paste made the filing session genuinely low-stress.”
— Mitchell Williams, Primary Tester    [UX Design]


“When I asked about the $0 AGI field instruction, the assistant didn't just repeat the guidance — it explained exactly why it applied only to the federal identity-verification field and not to the Illinois AGI field. That distinction mattered.”
— Mitchell Williams, Primary Tester    [Context Sensitivity]

Constructive Feedback — Areas for Improvement


“Session continuity is the biggest friction point. Each new conversation requires re-uploading or re-summarizing the document set. A persistent state layer that survives session boundaries would cut setup time by 80%.”
— Mitchell Williams, Primary Tester    [Session Continuity]


“The 1099-B still requires fully manual entry. Every RSU lot transaction has to be typed in by hand. Automated ingestion from a Schwab CSV or structured PDF would be a major quality-of-life improvement.”
— Mitchell Williams, Primary Tester    [Automation Gap]


“IRS citations were hit-or-miss. Sometimes I got a specific publication and section number; other times just a general reference. Consistent, specific citations for every recommendation would make audit defense much stronger.”
— Mitchell Williams, Primary Tester    [Citation Consistency]

Feedback Themes — Impact Analysis

Theme
Freq.
Impact
Effort
Recommended Action
Session state loss between conversations
High
High
High
Implement persistent knowledge base with auto-sync on session open
Manual 1099-B entry burden
High
High
High
Build Schwab CSV/PDF automated ingestion pipeline
Inconsistent IRS citation specificity
Medium
Medium
Low
Require citation template for every tax rule mentioned
Overgeneralization of field-specific instructions
Low
High
Low
Add field-scope validation: flag when an instruction is limited to one form or field
Software alert ambiguity (FreeTaxUSA)
Medium
High
Medium
Build alert classification system with risk level and independent analysis
NY W-2 Box 16 pre-fill risk
Low
High
Low
Add explicit multi-state wage allocation check to pre-filing checklist
No proactive deduction discovery
Medium
Medium
Medium
Add structured deduction interview to Phase 2 workflow

Key Insights for v2.0 Development
What Beta Testing Confirmed
The highest-value capability is independent verification of software-generated figures, not data entry assistance
Explicit uncertainty communication (inferred vs. sourced) is non-negotiable for tester trust
Pre-computation of filing values dramatically reduces cognitive load during the actual filing session
Session architecture (persistent state) matters as much as in-session reasoning quality

Behaviors to Preserve
Flag when figures are derived rather than directly sourced from uploaded documents
Treat each software alert as requiring independent analysis, not composite acceptance
Provide field-specific instructions with explicit scope boundaries
Lead with the most safety-critical issues before moving to optimization

Tax Assistant User Feedback Compilation  |  Confidential  |  Mitchell Williams  |  March 2026