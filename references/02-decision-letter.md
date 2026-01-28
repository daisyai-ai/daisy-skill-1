# Step 2: Clinical Review & Letter Generation

Present assessment for clinical review, capture agreement/disagreement, and generate the Status Validation Report.

## Purpose

This step provides a checkpoint where the hospital's clinical reviewer validates the AI's assessment before generating the final report. Their input creates an audit trail and flags cases needing deeper review.

## Prerequisites

- `waypoints/assessment.json` must exist from Step 1
- If missing, return to Step 1 (01-intake-assessment.md)

## Execution Checklist

```
Clinical Review & Letter Progress:
- [ ] 2.1 Load assessment from waypoint
- [ ] 2.2 Display clinical review checkpoint
- [ ] 2.3 Capture reviewer agreement/disagreement
- [ ] 2.4 Handle disagreement (if applicable)
- [ ] 2.5 Generate Status Validation Report
- [ ] 2.6 Write decision waypoint
- [ ] 2.7 Write final report (markdown)
- [ ] 2.8 Generate PDF
```

---

## 2.1 Load Assessment

Read `waypoints/assessment.json` and extract:
- Patient demographics
- Admission details
- Code validation results
- Medical necessity assessment
- Two-midnight compliance
- Documentation gaps
- Preliminary verdict

If file not found or corrupted, prompt user to re-run Step 1.

---

## 2.2 Display Clinical Review Checkpoint

Present structured summary using this format:

```
╔═══════════════════════════════════════════════════════════════════╗
║                    CLINICAL REVIEW CHECKPOINT                      ║
╠═══════════════════════════════════════════════════════════════════╣
║  Patient: [AGE][SEX] | [MRN]                                       ║
║  Current Status: [Inpatient/Observation]                           ║
║  Payer: [PAYER_NAME]                                               ║
╚═══════════════════════════════════════════════════════════════════╝

AI ASSESSMENT: [VERDICT] ([CONFIDENCE] Confidence)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[Brief explanation of what the AI found]

SEVERITY OF ILLNESS
• [Bullet list of key severity indicators with values]

INTENSITY OF SERVICES
• [Bullet list of services requiring hospital care]

TWO-MIDNIGHT COMPLIANCE
• Expected LOS: [VALUE] [✓/✗]
• Documented in MDM: [Yes/No] [✓/✗]
• Exceeds threshold: [Yes/No] [✓/✗]

CODE VALIDATION
• [Code] ([Description]) - [Valid/Invalid], [Billable/Non-billable]

DOCUMENTATION GAPS: [None identified / List of gaps]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Based on this data, the payer is [UNLIKELY/LIKELY] to deny this claim.
```

---

## 2.3 Capture Reviewer Agreement

Ask the clinical reviewer:

```
Do you agree with this assessment? (Agree/Disagree): ___
```

**If Agree**: Proceed to 2.5 (Generate Report)

**If Disagree**: Proceed to 2.4 (Handle Disagreement)

---

## 2.4 Handle Disagreement

If the reviewer disagrees, capture their reasoning:

```
Please describe your clinical reasoning for disagreement:
> ___

This case will be flagged for physician advisor review.
The report will note your clinical disagreement.

Proceed to generate report? (Y/N): ___
```

Record:
- `reviewer_agreement`: "DISAGREE"
- `reviewer_notes`: [their reasoning]
- `escalation_flag`: true

The report will include the disagreement and flag for escalation.

---

## 2.5 Generate Status Validation Report

Load the template from `assets/letter-template.md` and populate:

### Verdict-Specific Messaging

| Verdict | Screening Summary | Denial Risk |
|---------|-------------------|-------------|
| SUPPORTED | "Well-supported. Payer unlikely to deny." | LOW |
| AT RISK | "Borderline. Documentation may be insufficient." | MEDIUM |
| LIKELY DENIAL | "Documentation does not support current status." | HIGH |

### Section Generation

**Header Section**
- Patient demographics
- Admission date
- Current status
- Payer name

**Screening Result**
- Verdict with risk level
- Summary paragraph based on verdict type

**ICD-10 Code Validation**
- Table of codes with validation status
- Code status summary
- Recommendations for specificity improvements

**Medical Necessity Assessment**

*Severity of Illness*:
- Narrative explaining clinical indicators
- Table of indicators with values/thresholds
- Summary statement

*Intensity of Services*:
- Narrative explaining required services
- Table of services with evidence
- Summary statement

**Two-Midnight Rule Compliance** (Medicare only)
- Explanation of rule
- Compliance table
- Status determination

**Documentation Checklist**
- List documentation elements with [x] or [ ]
- Gap descriptions if any

**Clinical Reviewer Attestation**
- Reviewer agreement status
- Notes (if disagree)
- Timestamp

**Action Items**
- Required actions (based on verdict)
- Optional enhancements

### Action Items by Verdict

| Verdict | Required Actions | Optional |
|---------|-----------------|----------|
| SUPPORTED | None — proceed to claim submission | Code specificity improvements |
| AT RISK | Strengthen documentation before submission | Physician advisor review |
| LIKELY DENIAL | Consider status change OR escalate to physician advisor | Request addendum from provider |

---

## 2.6 Write Decision Waypoint

Save to `waypoints/decision.json`:

```json
{
  "request_id": "[from assessment]",
  "status": "decision_complete",
  "timestamp": "[ISO timestamp]",

  "assessment_verdict": "[from Step 1]",
  "final_verdict": "[same unless overridden]",

  "reviewer": {
    "agreement": "[AGREE/DISAGREE]",
    "notes": "[if any]",
    "timestamp": "[when reviewed]"
  },

  "escalation_required": [true/false],

  "output_file": "outputs/status-validation-report.md"
}
```

---

## 2.7 Write Final Report

Save the generated Status Validation Report to:
`outputs/status-validation-report.md`

---

## 2.8 Generate PDF

Convert markdown → HTML → PDF using pandoc + Chrome headless:

```bash
# Step 1: Markdown to HTML
pandoc outputs/status-validation-report.md -o outputs/status-validation-report.html

# Step 2: HTML to PDF via Chrome headless
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless --disable-gpu \
  --print-to-pdf=outputs/status-validation-report.pdf \
  outputs/status-validation-report.html
```

**Dependencies:** `pandoc` + Google Chrome (no LaTeX needed).

```bash
brew install pandoc  # if not installed
```

Update `decision.json` to include PDF path:

```json
{
  "output_files": {
    "markdown": "outputs/status-validation-report.md",
    "pdf": "outputs/status-validation-report.pdf"
  }
}
```

---

## Completion

After writing both files:

1. Confirm files were written successfully
1. Display completion message:

```text
╔═══════════════════════════════════════════════════════════════════╗
║                    STATUS DETERMINATION COMPLETE                   ║
╠═══════════════════════════════════════════════════════════════════╣
║  Final Verdict: [VERDICT]                                          ║
║  Denial Risk: [RISK_LEVEL]                                         ║
║  Reviewer Agreement: [AGREE/DISAGREE]                              ║
╚═══════════════════════════════════════════════════════════════════╝

Reports saved:
  • waypoints/decision.json
  • outputs/status-validation-report.md
  • outputs/status-validation-report.pdf

[If SUPPORTED]: Proceed to claim submission.
[If AT RISK]: Review documentation recommendations before submission.
[If LIKELY DENIAL]: Escalate to physician advisor or consider status change.
```

1. If escalation required, note: "This case has been flagged for physician advisor review."
