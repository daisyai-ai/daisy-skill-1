# Status Determination Skill

A Claude Code skill for evaluating hospital admission status (inpatient vs observation) against clinical evidence and payer criteria.

## What It Does

Screens inpatient/observation status decisions *before* claim submission—catching likely denials based on:
- ICD-10 code validation
- Medical necessity (severity of illness, intensity of services)
- Two-Midnight Rule compliance (Medicare)
- Documentation gaps

## Workflow

```
Step 1: Intake & Assessment
  → Parse ADT, labs, vitals, clinical notes
  → Apply clinical rubric
  → Generate preliminary verdict
  
[Clinical Review Checkpoint]

Step 2: Decision & Letter
  → Reviewer confirms/disputes assessment
  → Generate Status Validation Report (MD + PDF)
```

## Verdicts

| Verdict | Meaning | Action |
|---------|---------|--------|
| **SUPPORTED** | Evidence justifies status | Proceed to submission |
| **AT RISK** | Borderline, documentation gaps | Strengthen documentation |
| **LIKELY DENIAL** | Evidence doesn't support status | Change status or escalate |

## Usage

```
Run status determination on sample-data/patient-001
```

The skill will:
1. Parse the admission packet (ADT, labs, vitals, MDM note)
2. Validate ICD-10 codes via MCP tools
3. Assess against clinical rubric
4. Present checkpoint for clinical review
5. Generate a Status Validation Report

## Structure

```
├── SKILL.md              # Main skill orchestrator
├── rubric.md             # Clinical criteria thresholds
├── tools.md              # MCP tool integration patterns
├── references/
│   ├── 01-intake-assessment.md
│   └── 02-decision-letter.md
├── assets/
│   └── letter-template.md
├── examples/
│   └── case-001-sepsis-cap.md
└── sample-data/
    └── patient-00X/      # Synthetic test cases
```

## Requirements

- Claude Code with MCP tools enabled
- `ICD10_Codes` MCP server (for code validation)
- `pandoc` + Chrome (for PDF generation)

## License

MIT

---

Built by [Daisy AI](https://daisyai.com)
