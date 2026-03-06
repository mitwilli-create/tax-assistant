# Tax Assistant

A personal tax filing system built inside Claude Projects. Four-layer architecture using Claude as both the reasoning engine and the external state store. Built for a complex multi-state return under tight timing constraints — and designed to treat every software suggestion as a starting point for verification, not a conclusion.

**Built in:** One intensive session (~8 hours)  
**Key outcome:** Caught a $[NY_TAX_DIFFERENCE] state income tax error that the filing software pre-filled with confidence and then defended when questioned.

## The problem

Complex tax returns — multi-state income, RSU vesting events, 401k distributions, non-deductible IRA basis tracking — require holding a lot of interdependent information simultaneously while making decisions that are hard to reverse. Standard tax software hides the underlying logic and actively discourages questioning its outputs.

I wanted a system that understood the full picture, explained every decision in plain language, cited its sources, and treated software alerts as prompts for verification — not authoritative guidance.

## Architecture

**Four layers:**

**1. System prompt**
Defines the filing process (6 phases), requires plain-language explanation before every calculation, mandates IRS citation for every substantive claim, and enforces a `[PLACEHOLDER]` convention — if a value is unknown, it surfaces as unknown rather than being estimated.

**2. Knowledge base (18 files)**
Pre-loaded before filing day: W-2, all 1099s, prior returns, RSU vesting history, 401k and IRA records, pre-computed summary values, IRS reference docs. Everything the system needs is in the KB before the first question gets asked.

**3. Conversational memory**
Key derived facts — multi-state allocation calculations, basis tracking decisions, form-specific conclusions — tracked across sessions so context doesn't reset.

**4. User-provided corrections**
Explicit overrides logged to persistent memory. Prior errors don't carry forward.

## The $[NY_TAX_DIFFERENCE] catch

The filing software pre-filled a state income field with the full federal W-2 figure instead of the actual state-sourced income — a common error for RSU income allocated across multiple states based on work location during the vesting period.

- **Pre-filled figure:** ~$[W2_WAGES]
- **Correct state-sourced allocation:** ~$[NY_ALLOCATED_WAGES]
- **Tax liability difference:** ~$[NY_TAX_DIFFERENCE]

When I flagged it, the software generated an alert recommending I restore the pre-filled figure. The system identified the alert as incorrect, explained the multi-state allocation methodology, cited the relevant IRS guidance, and walked through the correct calculation.

**What made the catch possible:** The pre-computation layer had already calculated the correct allocation before filing day, so the discrepancy was immediately visible rather than buried in a UI flow designed to minimize friction.

## Key design decisions

**Why `[PLACEHOLDER]` instead of estimation?**
Tax software estimates. It fills fields with confident-looking numbers. The most dangerous failure mode is a confidently wrong number that passes visual inspection. The placeholder convention engineers against that — every unknown surfaces as an unknown.

**Why citation requirements?**
Requiring an IRS citation for every substantive claim creates a forcing function. The system can't assert something without sourcing it, and I can verify directly. It's the difference between "this is the rule" and "this is what I think the rule probably is."

**Why pre-compute everything before filing day?**
Filing day has enough cognitive load. Pre-computing all calculable values means the filing session is verification and execution, not calculation under pressure.

## Additional outcomes

- Identified non-deductible IRA basis issue (Form 8606) — recommended and executed Roth conversion
- Saved 10+ hours vs. CPA or traditional filing
- Filed on schedule despite tight post-surgery timing

## What's not here

Personal financial documents, actual tax figures, and account data aren't included. This repo is the architecture and design rationale only.

## Tech stack

- Claude Projects (Sonnet)
- Structured markdown knowledge base (18 files)
- FreeTaxUSA (filing interface)
