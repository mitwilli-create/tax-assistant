# 🧾 AI Tax Assistant — Claude Projects Implementation

> A practitioner-level demonstration of using Claude Projects as a structured, stateful tax preparation assistant for multi-state filers with complex compensation (RSUs, early IRA distributions, multi-state W-2 allocation).

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Feature Set](#feature-set)
3. [API / Integration Reference](#api--integration-reference)
4. [Tax Compliance Handling](#tax-compliance-handling)
5. [User Feedback & Iteration](#user-feedback--iteration)
6. [Deployment Instructions](#deployment-instructions)
7. [Testing Procedures](#testing-procedures)
8. [Future Development Plans](#future-development-plans)
9. [Use Cases](#use-cases)

---

## Architecture Overview

### System Design Philosophy

The Tax Assistant is built on **Claude Projects** (claude.ai), leveraging its persistent knowledge base as an external state store. Rather than relying on in-context memory within a single conversation, the system externalizes all document state, computed reference values, and filing-session decisions to a structured file set — enabling multi-session workflows with minimal cognitive load at execution time.

The guiding design principle: **pre-compute everything so that filing day is a 15-minute copy-paste session**, not a decision-making exercise.

---

### Four-Layer Knowledge Base Structure

```
📁 Claude Project Knowledge Base
│
├── Layer 1: Raw Source Documents        ← Ground truth; never modified
│   ├── 2025_W2_[EMPLOYER]_final.pdf
│   ├── 2025_1099DIV_[BROKERAGE]_dividends.pdf
│   ├── 2025_1099R_[BROKERAGE]_retirement.pdf
│   └── [1099-B when available]
│
├── Layer 2: Historical Reference Data   ← Context and trend analysis
│   ├── RSU_Vesting_History.xlsx
│   ├── Brokerage_History.xlsx
│   ├── Paycheck_History.xlsx
│   ├── Income_History.xlsx
│   ├── Tax_History.xlsx
│   ├── 401k_History.xlsx
│   └── Investment_Performance.xlsx
│
├── Layer 3: Computed Reference Docs     ← Pre-calculated; reduces live computation
│   ├── 2025_Tax_Filing_Prefill.md       ← Copy-paste-ready field values
│   ├── 2025_Tax_Master_Summary.xlsx     ← Consolidated view across all forms
│   ├── Financial_KPIs.md               ← Dashboard of key metrics
│   └── Net_Worth_Tracker.xlsx
│
└── Layer 4: Status & Context Docs      ← Decision log and issue tracking
    ├── 2024_IRS_Return_Status.md       ← Prior-year issue documentation
    └── 2025_Document_Inventory.xlsx    ← Checklist of received documents
```

**Why this structure matters:** Each layer serves a distinct purpose. Raw documents are never mutated. Computed reference docs are regenerated when source documents change. Status docs capture edge cases and workarounds (e.g., the $0 AGI e-file workaround) so they survive across sessions without re-derivation.

---

### System Prompt Architecture

The system prompt encodes the assistant persona across six behavioral domains:

| Domain | Implementation |
|---|---|
| **Educational mode** | Every concept explained in plain language before application |
| **Citation requirements** | Mandatory IRS publication citations for all rules |
| **Conservative posture** | Default to safer interpretations; flag audit risk explicitly |
| **Interview cadence** | One question at a time; summarize every 5–7 questions |
| **Phase gating** | Six structured phases (Document Verification → W-4 Optimization) |
| **Response format** | Structured template: What it is / How it affects you / IRS Source / Watch out for |

---

### Multi-State Filing Architecture

The tax situation requires nonresident returns for two states alongside the federal return:

```
Federal (Form 1040)
    ↓ credits flow to
Illinois Nonresident (IL-1040-NR)    ← W-2 Box 16: $[IL_WAGES] | Withheld: $[IL_WITHHELD]
New York Nonresident (IT-203)        ← W-2 Box 17: $[NY_WITHHELD] withheld | Allocated: ~$[NY_ALLOCATED_WAGES]
Washington State                     ← No income tax; current domicile
```

**Critical NY allocation insight:** Box 16 for NY shows $[W2_WAGES] (full W-2 wages) — but this is a W-2 formatting artifact, not the taxable NY-sourced amount. The correct NY-sourced wage figure (~$[NY_ALLOCATED_WAGES]) is back-calculated from the $[NY_WITHHELD] withholding amount. Accepting the W-2 face value would generate ~$[NY_TAX_DIFFERENCE] in excess NY tax liability.

---

## Feature Set

### 1. Stateful Multi-Session Continuity

**What it does:** Persistent knowledge base files carry all context across sessions, so each conversation starts fully informed — no re-uploading documents, no re-explaining the situation.

**How it contributes:** Eliminates the #1 pain point in complex AI-assisted workflows: context loss between sessions. The project functions like a shared drive + briefing document rolled into one.

---

### 2. Pre-Computed Filing Prefill (`2025_Tax_Filing_Prefill.md`)

**What it does:** All key tax fields are computed and formatted before the filing session begins. The prefill document contains copy-paste-ready values for every line that requires a number.

**Structure:**
```
W-2 Wages (Line 1):           $[W2_WAGES]
Federal Tax Withheld (W-2):    $[FED_WITHHELD]
Ordinary Dividends:               $[DIVIDENDS]
Qualified Dividends:              $[DIVIDENDS]
IRA Distribution (Taxable):       $[IRA_DIST]
IRA Tax Withheld:                  $[IRA_PENALTY]
Total Federal Tax Withheld:    $[TOTAL_FED_WITHHELD]
```

**How it contributes:** Reduces filing-day cognitive load to near zero. The filer becomes an executor of a pre-verified plan rather than a decision-maker under time pressure.

---

### 3. Mandatory Citation Engine

**What it does:** Every tax rule invoked by the assistant is tagged with an IRS publication, form instruction, or IRC section reference.

**Citation format:**
- `[IRS Publication 590-B, Chapter 1]` — IRA distributions
- `[IRS Form 1099-R Instructions, Box 7]` — Distribution codes
- `[IRC §72(t)]` — 10% early withdrawal penalty
- `[IRS Publication 519]` — Multi-state allocation rules

**How it contributes:** Converts the assistant from an opaque oracle into a verifiable reference system. Every claim can be independently confirmed. This is especially critical for edge cases like multi-state RSU allocation, where incorrect application of a rule carries thousands of dollars of consequence.

---

### 4. Software Alert Verification Protocol

**What it does:** When FreeTaxUSA (or any filing software) generates an alert or recommendation, the assistant evaluates it independently rather than deferring to the software's framing.

**Documented example — NY Wage Alert:**
- Software pre-filled: $[W2_WAGES] as NY wages
- Software issued alert recommending reversion to $[W2_WAGES]
- Assistant analysis: Withholding of $[NY_WITHHELD] implies ~[RATE]% effective rate on $20,000, or ~[RATE]% on $[W2_WAGES] — the latter is implausible
- Correct entry confirmed: ~$[NY_ALLOCATED_WAGES] (back-calculated from withholding)
- Stakes: Accepting the alert's recommendation would have generated ~$[NY_TAX_DIFFERENCE] in excess NY tax

**How it contributes:** Treats software alerts as hypotheses, not authorities. Each alert gets its own independent analysis.

---

### 5. Field-Scoped Instruction Management

**What it does:** Tracks which instructions apply to which specific fields, preventing overgeneralization.

**Documented example — AGI Field Disambiguation:**
- Instruction: Enter $0 for prior-year AGI (federal e-file identity verification field)
- Reason: 2024 return not yet in IRS systems; $0 is the correct workaround per IRS guidance
- Scope: **Federal identity verification field only**
- Illinois AGI field: Requires actual AGI figure for tax calculation purposes
- Risk mitigated: Without this distinction, the $0 instruction applied to the wrong field would corrupt the IL state return

**How it contributes:** Prevents a common failure mode in complex, multi-form tax workflows where a workaround for one field gets cargo-culted to similar-looking fields.

---

### 6. Uncertainty Flagging

**What it does:** The assistant explicitly flags when a figure is derived by inference (e.g., back-calculated from withholding) versus lifted directly from a primary document.

**How it contributes:** Creates a clear epistemic hierarchy. The filer knows which values are document-verified and which are computed estimates, enabling appropriate confidence calibration.

---

### 7. Six-Phase Interview Protocol

| Phase | Purpose |
|---|---|
| **1. Document Verification** | Confirm all source documents are present and accurately extracted |
| **2. Income Summary** | Walk through each income source with tax treatment explanation |
| **3. Deduction Discovery** | Targeted questioning to surface commonly missed deductions |
| **4. Deduction Strategy** | Standard vs. itemized analysis with the math shown |
| **5. Form Walkthrough** | Line-by-line Form 1040 with source attribution |
| **6. W-4 Optimization** | Withholding modeling to achieve refund target |

---

### 8. Financial KPI Dashboard

**What it does:** Synthesizes multi-year financial data into a single-page dashboard covering income trajectory, 401(k) growth, effective tax rates, net worth components, and millionaire-milestone projections.

**Key metrics tracked:**
- 8-year income CAGR: [INCOME_CAGR]%
- 2025 effective federal rate: [EFF_TAX_RATE]%
- 401(k) balance trajectory (Q2 2023 → Q4 2025)
- Net worth milestone progress ($[NET_WORTH_1] by 2027, $[NET_WORTH_2] by 2032)

---

## API / Integration Reference

> **Note:** This project runs natively on Claude Projects (claude.ai) and does not expose HTTP API endpoints in the traditional sense. The sections below document the integration surface for developers who want to replicate this architecture via the Anthropic Messages API or build tooling around it.

---

### Anthropic Messages API — Core Endpoint

```
POST https://api.anthropic.com/v1/messages
```

**Authentication:**
```
x-api-key: YOUR_ANTHROPIC_API_KEY
anthropic-version: 2023-06-01
Content-Type: application/json
```

**Minimal request body:**
```json
{
  "model": "claude-opus-4-5",
  "max_tokens": 4096,
  "system": "<SYSTEM_PROMPT_HERE>",
  "messages": [
    {
      "role": "user",
      "content": "Let's verify my W-2 data. I'm looking at Box 1..."
    }
  ]
}
```

**Document injection (PDF source documents):**
```json
{
  "role": "user",
  "content": [
    {
      "type": "document",
      "source": {
        "type": "base64",
        "media_type": "application/pdf",
        "data": "<BASE64_ENCODED_PDF>"
      }
    },
    {
      "type": "text",
      "text": "Please extract all key fields from this W-2."
    }
  ]
}
```

**Response format:**
```json
{
  "id": "msg_...",
  "type": "message",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "**W-2 Wages (Box 1)**\n📋 What it is: ..."
    }
  ],
  "model": "claude-opus-4-5",
  "stop_reason": "end_turn",
  "usage": {
    "input_tokens": 1842,
    "output_tokens": 634
  }
}
```

---

### Knowledge Base Integration Pattern

For API-based implementations that replicate the Projects knowledge base:

```python
import anthropic
import base64
from pathlib import Path

def load_knowledge_base(doc_paths: list[str]) -> list[dict]:
    """
    Convert knowledge base files into Anthropic document blocks.
    Supports PDF (binary) and markdown/text (text) formats.
    """
    documents = []
    for path in doc_paths:
        p = Path(path)
        if p.suffix == ".pdf":
            with open(p, "rb") as f:
                data = base64.standard_b64encode(f.read()).decode("utf-8")
            documents.append({
                "type": "document",
                "source": {
                    "type": "base64",
                    "media_type": "application/pdf",
                    "data": data
                },
                "title": p.stem
            })
        elif p.suffix in [".md", ".txt"]:
            with open(p, "r") as f:
                content = f.read()
            documents.append({
                "type": "text",
                "text": f"### {p.stem}\n\n{content}"
            })
    return documents

KNOWLEDGE_BASE_PATHS = [
    "2025_Tax_Filing_Prefill.md",
    "2024_IRS_Return_Status.md",
    "Financial_KPIs.md",
    "2025_W2_[EMPLOYER]_final.pdf",
    "2025_1099DIV_[BROKERAGE]_dividends.pdf",
    "2025_1099R_[BROKERAGE]_retirement.pdf",
]

client = anthropic.Anthropic()
kb_blocks = load_knowledge_base(KNOWLEDGE_BASE_PATHS)

response = client.messages.create(
    model="claude-opus-4-5",
    max_tokens=4096,
    system=SYSTEM_PROMPT,
    messages=[
        {
            "role": "user",
            "content": kb_blocks + [
                {"type": "text", "text": "Please begin Phase 1: Document Verification."}
            ]
        }
    ]
)
```

---

### Filing Software Integration (FreeTaxUSA Reference)

The Tax Assistant is designed to operate alongside FreeTaxUSA as the filing platform. There is no direct API integration; the assistant provides field values that the user enters manually into the software. The key integration points are:

| Filing Software Field | Source in Assistant | Notes |
|---|---|---|
| W-2 Box 1 wages | `2025_Tax_Filing_Prefill.md` | $[W2_WAGES] |
| Federal tax withheld | Prefill doc | $[TOTAL_FED_WITHHELD] (W-2 + 1099-R) |
| IL state wages | W-2 Box 16 (IL row) | $[IL_WAGES] — enter directly |
| NY state wages | Back-calculated | ~$[NY_ALLOCATED_WAGES] — NOT W-2 Box 16 |
| Prior-year AGI (federal) | IRS workaround | $0 (2024 return unprocessed) |
| Prior-year AGI (IL field) | Actual figure | Real AGI — different field, different rule |

---

## Tax Compliance Handling

### Compliance Framework

The assistant operates under a conservative compliance posture with three tiers of authority:

1. **Primary authority:** IRS publications, form instructions, IRC sections (cited inline)
2. **Secondary authority:** State-specific guidance (IL Department of Revenue, NY DTF)
3. **Tertiary context:** Professional judgment flags — situations where a CPA consultation is recommended

### Key Compliance Rules Applied

**Federal:**
- Standard deduction applied (2025: $15,000 single) [IRS Rev. Proc. 2024-40]
- Qualified dividends taxed at preferential rates (0/15/20%) [IRC §1(h)]
- Early IRA distribution 10% penalty flagged [IRC §72(t); Form 5329]
- RSU income included in W-2 Box 1 at FMV on vesting date [IRC §83; IRS Pub. 525]
- FICA wage base cap applied ($176,100 for 2025 social security) [IRS Notice 2024-80]

**Illinois (Nonresident):**
- IL nonresident files IL-1040-NR [86 Ill. Adm. Code 100.3020]
- Wages sourced to IL based on actual days worked in state
- IL Box 16 wages ($[IL_WAGES]) accepted as reported by employer

**New York (Nonresident):**
- NY nonresident files IT-203 [NY Tax Law §631]
- NY source income = wages earned while physically present in NY or attributable to NY work
- W-2 Box 16 (NY) shows $[W2_WAGES] — treated as formatting artifact; actual sourced amount ~$[NY_ALLOCATED_WAGES]
- Withholding-implied allocation used as primary derivation method

### Audit Risk Assessment Protocol

For any position that deviates from a straightforward reading of a tax form, the assistant applies a three-factor audit risk assessment:

1. **Dollar magnitude** — Is the difference material enough to trigger scrutiny?
2. **Supportability** — Can the position be documented with a clear paper trail?
3. **Precedent** — Is there IRS guidance or case law supporting the position?

The NY wage allocation is assessed as **low audit risk** because:
- The withholding-implied amount ($[NY_ALLOCATED_WAGES] at ~7% effective rate) is entirely plausible
- [EMPLOYER]'s payroll system generated the withholding based on actual NY work performed
- The filing will include full W-2 documentation showing the withholding basis

---

## User Feedback & Iteration

### Key Feedback Themes

**1. Alert skepticism as a feature, not a bug**

Initial interactions with FreeTaxUSA alerts revealed a tendency to defer to software recommendations. The project documentation now explicitly codifies "software alerts require independent analysis" as a standing principle. Each alert is treated as a hypothesis requiring its own verification pass.

**2. Field specificity matters more than expected**

The $0 AGI workaround nearly propagated to the wrong field. This surfaced the need for explicit field-scope tracking — instructions are now annotated with which specific form and field they apply to, preventing cargo-culting.

**3. Uncertainty flagging builds trust**

Users (the primary tester included) noted that explicit flagging of inferred vs. document-verified figures — rather than presenting all values with equal confidence — significantly increased trust in the outputs. "I know when to double-check" was cited as a key value.

**4. Pre-computation reduces filing-day anxiety**

Designing the workflow so that all decisions are made before filing day, and filing day is purely execution, was cited as the most practically valuable design choice. This was especially relevant given the recovery context (post-surgery filing window).

### Iteration History

| Version | Change | Trigger |
|---|---|---|
| v1.0 | Basic document extraction and field mapping | Initial setup |
| v1.1 | Added NY allocation back-calculation | W-2 Box 16 NY artifact discovered |
| v1.2 | Added field-scope annotation to all instructions | $0 AGI near-miss on IL field |
| v1.3 | Added uncertainty flagging convention | Trust calibration feedback |
| v1.4 | Codified software alert verification protocol | FreeTaxUSA NY alert incident |
| v1.5 | Added 2024 IRS status tracking doc | 2024 return processing delay discovered |

---

## Deployment Instructions

### Option A: Claude Projects (Recommended — No Code Required)

1. **Create a Claude Project** at claude.ai
2. **Upload knowledge base files** to the Project's file store (18 files max recommended):
   - All source tax documents (W-2, 1099s as PDFs)
   - Historical Excel files (RSU history, brokerage history, etc.)
   - Pre-computed reference docs (prefill markdown, KPI dashboard)
   - Status/context docs (IRS status, document inventory)
3. **Set the system prompt** — paste the Tax Assistant persona from this repo's `system_prompt.md`
4. **Start a conversation** — the knowledge base is automatically available in every session

**Advantages:** Zero infrastructure, persistent across sessions, no API key management, native PDF understanding.

---

### Option B: API-Based Implementation

**Prerequisites:**
```
- Python 3.11+
- anthropic>=0.40.0
- python-dotenv
```

**Environment variables:**
```bash
ANTHROPIC_API_KEY=sk-ant-...
MODEL=claude-opus-4-5
MAX_TOKENS=4096
KNOWLEDGE_BASE_DIR=./knowledge_base/
```

**Installation:**
```bash
git clone https://github.com/mitwilli-create/tax-assistant
cd tax-assistant
pip install -r requirements.txt
cp .env.example .env
# Add your ANTHROPIC_API_KEY to .env
python main.py
```

**Directory structure:**
```
tax-assistant/
├── main.py                      # Entry point
├── system_prompt.md             # Tax assistant persona
├── knowledge_base/
│   ├── source_docs/             # Raw PDFs (W-2, 1099s)
│   ├── historical/              # Excel history files
│   ├── computed/                # Prefill docs, KPI dashboard
│   └── status/                  # IRS status, inventory
├── utils/
│   ├── document_loader.py       # KB file → API document blocks
│   └── citation_checker.py     # Validates IRS citations in responses
└── requirements.txt
```

**Dependencies (`requirements.txt`):**
```
anthropic>=0.40.0
python-dotenv>=1.0.0
openpyxl>=3.1.0           # Excel file reading
pypdf>=4.0.0              # PDF text extraction for preprocessing
rich>=13.0.0              # Terminal formatting
```

---

## Testing Procedures

### 1. Document Extraction Accuracy Tests

For each source document, verify that extracted values match the primary document:

```python
EXTRACTION_TEST_CASES = [
    {"field": "W2_Box1_Wages", "expected": [W2_WAGES], "source": "W-2 Box 1"},
    {"field": "W2_Box2_FedWithheld", "expected": [FED_WITHHELD], "source": "W-2 Box 2"},
    {"field": "W2_IL_Wages", "expected": [IL_WAGES], "source": "W-2 Box 16 IL"},
    {"field": "W2_NY_Withholding", "expected": [NY_WITHHELD], "source": "W-2 Box 17 NY"},
    {"field": "DIV_1a_OrdinaryDiv", "expected": [DIVIDENDS], "source": "1099-DIV Box 1a"},
    {"field": "IRA_Gross", "expected": [IRA_DIST], "source": "1099-R Box 1"},
    {"field": "IRA_FedWithheld", "expected": [IRA_PENALTY], "source": "1099-R Box 4"},
]
```

### 2. Tax Calculation Verification Tests

Run parallel calculations against IRS tax tables to verify computed estimates:

```python
# 2025 Single filer tax brackets
BRACKETS_2025 = [
    (11925, 0.10),
    (48475, 0.12),
    (103350, 0.22),
    (197300, 0.24),
    (250525, 0.32),
    (626350, 0.35),
    (float('inf'), 0.37),
]

def verify_tax_estimate(taxable_income: float, assistant_estimate: float, tolerance: float = 100):
    computed = calculate_tax(taxable_income, BRACKETS_2025)
    delta = abs(computed - assistant_estimate)
    assert delta <= tolerance, f"Tax estimate off by ${delta:.2f}"
```

### 3. Multi-State Allocation Consistency Test

Verify that state-allocated wages do not exceed federal wages and that allocation methodology is internally consistent:

```python
def test_state_allocation_consistency(federal_wages, il_wages, ny_wages_allocated):
    assert il_wages <= federal_wages, "IL wages cannot exceed federal wages"
    assert ny_wages_allocated <= federal_wages, "NY wages cannot exceed federal wages"
    # Verify NY back-calculation consistency
    ny_withholding = [NY_WITHHELD]
    implied_rate = ny_withholding / ny_wages_allocated
    assert 0.04 <= implied_rate <= 0.10, f"Implied NY rate {implied_rate:.1%} is implausible"
```

### 4. Citation Presence Tests

Verify that all tax rule assertions include IRS citations:

```python
import re

def test_citations_present(response_text: str):
    # Every paragraph containing a tax rule should cite a source
    rule_patterns = [r'\d+%', r'deduct', r'withheld', r'taxable', r'penalty']
    citation_pattern = r'\[IRS|IRC §|Publication \d+'
    
    for pattern in rule_patterns:
        if re.search(pattern, response_text, re.IGNORECASE):
            assert re.search(citation_pattern, response_text), \
                f"Tax rule referenced without citation: pattern '{pattern}'"
```

### 5. Regression Tests — Known Edge Cases

| Test | Input | Expected | Fail condition |
|---|---|---|---|
| NY wage override | W-2 Box 16 NY = $[W2_WAGES] | System flags; uses ~$[NY_ALLOCATED_WAGES] | System accepts $[W2_WAGES] |
| AGI field scope | $0 AGI instruction | Applied to federal field only | Applied to IL AGI field |
| Software alert | FreeTaxUSA alert to revert NY wages | Alert analyzed independently | Alert accepted without analysis |
| IRA penalty check | 1099-R code 1 | 10% penalty flagged | Penalty missed |

---

## Future Development Plans

### Near-Term (Q2 2026)

- **1099-B Integration:** Once the [BROKERAGE] 1099-B arrives (~Feb 2026), add Form 8949 / Schedule D prefill generation to the pipeline. This includes wash sale adjustment tracking and RSU cost basis verification.
- **Post-filing reconciliation doc:** After e-filing, generate a "what we filed" summary document capturing final values for all three returns (federal, IL, NY) for the project knowledge base.
- **2024 return resolution tracking:** Update `2024_IRS_Return_Status.md` once the return processes; verify capital gains reporting matches the filed amounts.

### Medium-Term (2026 Tax Year Planning)

- **W-4 optimization workflow:** Model 2026 withholding scenarios based on expected RSU vest schedule. Goal: achieve ~$[REFUND_TARGET] refund (down from current ~$[OVER_WITHHELD] over-withholding).
- **Covered call income tracking:** As the covered call strategy generates premium income, add a tracking layer for short-term capital gains from options expiration/assignment.
- **Quarterly estimated tax assessment:** Evaluate whether side income (if any consulting materializes) triggers estimated payment requirements.

### Long-Term / Community Contributions

- **Template generalization:** Extract the persona and KB structure into a reusable template for other high-compensation employees with RSU-heavy W-2s and multi-state exposure.
- **State tax module library:** Build modular state-specific guidance documents for common nonresident scenarios (CA, NY, IL, MA) that can be added to the knowledge base on demand.
- **Citation validation tool:** A lightweight script that parses assistant responses and checks that cited IRS publication sections actually exist and say what they're claimed to say.
- **Document change detection:** A utility that compares year-over-year tax documents and auto-flags differences that require new analysis (e.g., new Box 12 codes, changed state wage amounts).

---

## Use Cases

### Use Case 1: Multi-State W-2 Wage Allocation

**Situation:** Taxpayer receives a W-2 showing $[W2_WAGES] as NY state wages (Box 16), but only worked in NY briefly in 2025.

**Problem without assistant:** Filing software pre-fills NY wages at $[W2_WAGES]. Taxpayer may not recognize this as a formatting artifact rather than an accurate allocation. Filing with this figure generates ~$[NY_TAX_DIFFERENCE] in excess NY tax.

**Interaction flow:**
```
User:      FreeTaxUSA is showing $[W2_WAGES] for NY wages. Is that right?
```
User:      FreeTaxUSA is showing $[W2_WAGES] for NY wages. Is that right?
Assistant: That figure looks like a W-2 formatting artifact. Here's the analysis:
           - NY withholding (Box 17): $[NY_WITHHELD]
           - If NY wages were truly $[W2_WAGES], the implied rate would be [RATE]% — implausibly low
           - Back-calculating: $[NY_WITHHELD] / ~[NY_EFF_RATE]% effective rate = ~$[NY_ALLOCATED_WAGES] actual NY-sourced wages
           - Recommendation: Enter ~$[NY_ALLOCATED_WAGES] for NY wages, not $[W2_WAGES]
           [IRS Pub. 519; NY IT-203 Instructions]

User:      FreeTaxUSA just sent an alert recommending I revert to $[W2_WAGES]. Should I?
Assistant: No — the alert should be evaluated independently. The alert is recommending the
           W-2 face value, but as we established, that's a payroll formatting convention
           not an allocation. Your withholding of $[NY_WITHHELD] is the primary evidence of
           actual NY-sourced income. The ~$[NY_ALLOCATED_WAGES] figure remains the correct entry.
```

**Value delivered:** ~$[NY_TAX_DIFFERENCE] in avoided excess NY tax liability. The key behavior is treating the software alert as a hypothesis, not an authority.

---

### Use Case 2: AGI Field Disambiguation (E-file Workaround)

**Situation:** 2024 federal return is unprocessed by IRS. E-file requires prior-year AGI for identity verification. Standard workaround: enter $0.

**Problem without assistant:** The $0 instruction is correct for the federal identity-verification field, but a user might reasonably apply it to the Illinois AGI field — which uses the real figure for actual tax calculations.

**Interaction flow:**
```
User:      You said to enter $0 for prior-year AGI. IL is also asking for AGI. Same answer?
Assistant: Different field, different rule. Let me be precise:
           - Federal e-file identity verification field: Enter $0
             Reason: IRS has not processed your 2024 return, so $0 is the
             correct signal that the prior-year return is not in their system
           - Illinois AGI field: Enter your actual 2024 AGI
             Reason: IL uses this figure for tax calculation, not identity
             verification. Entering $0 here would corrupt your IL return.
           [2024_IRS_Return_Status.md; IL-1040-NR Instructions Line 1]
```

**Value delivered:** Prevented a field-scope error that would have produced an incorrect Illinois return.

---

### Use Case 3: Low-Cognitive-Load Filing Day

**Situation:** Filer is in post-surgery recovery. Needs to file three returns (federal, IL, NY) with minimal decision-making.

**Problem without assistant:** A complex multi-state filing with RSUs, multi-state allocation questions, and an IRS processing issue in the background would normally require significant active problem-solving during the filing session.

**Solution architecture:**
- All field values pre-computed and stored in `2025_Tax_Filing_Prefill.md`
- All edge cases documented with explicit resolutions in status docs
- All software alerts pre-analyzed with "correct answer" confirmed
- Filing session becomes copy-paste execution, not decision-making

**Result:** Estimated filing session time: 15 minutes across three returns.

---

### Use Case 4: RSU Cost Basis Verification

**Situation:** RSU vesting creates ordinary income included in W-2 Box 1. When shares are subsequently sold, the cost basis should equal the FMV at vesting (already taxed as income). If the brokerage reports incorrect cost basis, the filer pays capital gains tax on income already taxed as ordinary income.

**Interaction flow:**
```
User:      How do I know if [BROKERAGE] reported the right cost basis on my 1099-B?
Assistant: RSU cost basis errors are one of the most common
           and expensive mistakes in tech employee tax filings. Here's the check:
           
           1. Find your RSU vesting confirmations (from [BROKERAGE] or [EMPLOYER]'s equity portal)
           2. For each lot sold, the cost basis should equal:
              (# shares vested) x (closing price on vest date)
           3. This amount should already appear in your W-2 Box 1 wages
           4. If [BROKERAGE] shows $0 or a lower basis, your 1099-B is incorrect
           5. You can still file correctly by entering the right basis on Form 8949
              and noting "basis reported to IRS is incorrect"
           
           [IRS Publication 525, "Restricted Property"; Form 8949 Instructions]
           
           Once your 1099-B arrives, bring it back here and we'll verify
           each lot against your RSU_Vesting_History.xlsx.
```

**Value delivered:** Prevented double-taxation on RSU income by ensuring correct cost basis is reported even when [BROKERAGE]'s automated reporting may understate it.

---

### Use Case 5: Financial KPI Synthesis

**Situation:** Filer wants a consolidated view of financial health across income growth, retirement progress, and net worth trajectory — not just tax filing.

**Problem without assistant:** Data is scattered across 8+ Excel files covering paycheck history, 401(k) contributions, brokerage statements, and RSU vesting records.

**Output:** The `Financial_KPIs.md` document synthesizes:
- 8-year income CAGR ([INCOME_CAGR]% from $[PRIOR_INCOME] to $[W2_WAGES])
- 401(k) trajectory with projected balance at retirement
- Effective tax rate trend ([AVG_EFF_RATE]% average; [EFF_TAX_RATE]% in 2025)
- Net worth milestone timeline ($[NET_WORTH_1] by 2027, $[NET_WORTH_2] by ~2032–2035)
- Washington state tax savings vs. prior IL residency (~$[STATE_TAX_SAVINGS]+/year)

**Value delivered:** A single-page financial dashboard derived from the same documents used for tax preparation, with no additional data entry required.

---

## Contributing

Contributions are welcome in the following areas:

- **State tax modules** — Guidance documents for additional nonresident states (CA, MA, NJ, etc.)
- **Document parsers** — Utilities to extract structured data from common tax PDF formats
- **Citation validators** — Tools to verify IRS publication references
- **KB templates** — Generalized knowledge base structures for other tax situations (self-employed, rental income, foreign income, etc.)

Please open an issue before submitting a PR for substantive changes.

---

## Disclaimer

This project is for educational and demonstration purposes. It documents one taxpayer's approach to using AI tools in tax preparation. Nothing in this repository constitutes tax, legal, or financial advice. Consult a licensed CPA or tax attorney for advice specific to your situation.

Tax law changes frequently. All IRS citations reference rules as understood in early 2026. Verify current rules at irs.gov before relying on any guidance here.

---

*Built with Claude Projects (Anthropic) | Tax Year 2025 | Last Updated: March 2026*
