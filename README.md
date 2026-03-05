# Tax Assistant — Claude-Powered Filing System

A personal tax filing assistant built inside Claude Projects. Four-layer architecture using Claude as an external state store, structured knowledge base, and guided filing copilot for a complex multi-state return.

**Built in:** Single intensive session (~8 hours)  
**Filed:** Successfully, on schedule  
**Key outcome:** Caught a $19,000 state income tax error pre-filed by the software

---

## Problem

Complex tax returns — multi-state income, RSU vesting events, 401k distributions, non-deductible IRA basis tracking — require holding a large amount of interdependent information simultaneously while making consequential decisions under time pressure. Standard tax software presents UI flows that obscure the underlying logic and actively discourage questioning their outputs.

The goal: build a system that understands the full picture, explains every decision in plain language, cites authoritative sources, and treats every software suggestion as a starting point for verification — not a conclusion.

## Architecture

### Four-Layer Context Design

**Layer 1: System Prompt**
Custom system prompt defining:
- 6-phase filing process (document intake → deduction interview → line-by-line guidance → state allocation → verification → filing)
- Educational mode (every concept explained before applied)
- Citation requirements (IRS publication cited for every substantive claim)
- Conservative calculation approach
- `[PLACEHOLDER]` convention — if a value is unknown, insert a placeholder rather than estimate

**Layer 2: Knowledge Base (18 files)**
Pre-loaded structured documents:
- Income documents (W-2, all 1099 forms)
- Prior year returns and historical income data
- RSU vesting history (multi-year)
- 401k and IRA account history
- Pre-computed summary values
- IRS status and reference documents

**Layer 3: Conversational Memory**
Key derived facts tracked across sessions — multi-state allocation calculations, basis tracking decisions, form-specific conclusions — so context isn't lost between sessions.

**Layer 4: User-Provided Memory Edits**
Explicit corrections and overrides logged to persistent memory to prevent prior errors from persisting into future sessions.

### Pre-Computation Pattern

All calculable values were pre-computed before the filing session and loaded into the KB. Filing day becomes a verification and execution task rather than a calculation task. This reduces cognitive load at the moment of highest consequence.

## Key Design Decisions

**Why `[PLACEHOLDER]` over estimation?**
Tax software estimates. It fills fields with confident-looking numbers. The most dangerous failure mode in tax filing is a confidently wrong number that passes visual inspection. The placeholder convention forces every unknown to surface as an unknown — it engineers against the system's instinct to look finished.

**Why citation requirements?**
IRS publications are authoritative. Requiring a citation for every substantive claim creates a forcing function: the system can't assert something without sourcing it, and the user can verify directly. This catches the difference between "this is the rule" and "this is what I think the rule probably is."

**Why 18 files in the KB instead of pasting documents as needed?**
Persistent, pre-structured context means the system has the full picture at all times. Pasting documents ad hoc creates gaps, misses cross-document relationships, and requires manual tracking of what's been shared. A pre-loaded KB means the system can surface relevant context proactively.

## Critical Catch: $19,000 State Tax Error

Tax software pre-filled a state income field with the full W-2 Box 1 federal wages figure rather than the actual state-sourced income — a common error for employees with income allocated across multiple states based on work location during the vesting period.

The correct state-sourced allocation was ~$10,000. The pre-filled figure was ~$335,000.

The system flagged this discrepancy during the line-by-line review phase. The software then generated an alert recommending the pre-filled figure be restored. The system identified this alert as incorrect, explained the multi-state allocation methodology, cited the relevant IRS guidance, and walked through the correct calculation.

Filing with the software's pre-filled figure would have generated approximately $19,000 in excess state tax liability.

**What made this catch possible:**
- The pre-computation layer had already calculated the correct allocation before filing day
- The citation requirement meant the correct methodology was sourced and defensible
- The system was designed to treat software alerts as prompts for verification, not authoritative guidance

## Additional Outcomes

- Identified non-deductible IRA basis issue (Form 8606) — recommended and executed Roth conversion
- Saved 10+ hours vs. traditional filing or CPA consultation
- Filed on a tight post-surgery timeline using pre-surgery architecture

## What's Not In This Repo

The production system contained personal financial documents, actual tax figures, and individual IRS account data — none of which are included here. This repo documents the architecture, design decisions, and system design only.

## Tech Stack

- Claude Projects (Sonnet)
- Structured markdown knowledge base
- FreeTaxUSA (filing platform)
