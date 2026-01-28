# Step 1: Intake & Assessment

Parse clinical data and evaluate against payer denial risk factors.

## Purpose

This step extracts structured data from the admission packet and applies the clinical rubric to generate a preliminary verdict. The output is saved to `waypoints/assessment.json` for Step 2.

## Input Requirements

Provide paths to 4 data files:
- **ADT message** (demographics, diagnoses, payer, admission type)
- **ORU labs** (lab results with reference ranges)
- **ORU vitals** (vital signs over time)
- **MDM note** (clinical narrative from provider)

## Execution Checklist

```
Intake & Assessment Progress:
- [ ] 1.1 Parse ADT message
- [ ] 1.2 Parse ORU labs
- [ ] 1.3 Parse ORU vitals
- [ ] 1.4 Parse MDM note
- [ ] 1.5 Validate ICD-10 codes
- [ ] 1.6 Assess Severity of Illness
- [ ] 1.7 Assess Intensity of Services
- [ ] 1.8 Check Two-Midnight Rule compliance
- [ ] 1.9 Identify documentation gaps
- [ ] 1.10 Generate preliminary verdict
- [ ] 1.11 Write waypoint file
```

---

## 1.1 Parse ADT Message

Extract from ADT:

| Field | Location | Example |
|-------|----------|---------|
| Patient ID (MRN) | PID.patient_id | MRN-998877 |
| Age | PID.age_calculated or from birth_date | 78 |
| Sex | PID.sex | M |
| Admission Type | PV1.patient_class | I (Inpatient) / O (Observation) |
| Admission Date | event_timestamp | 2023-10-27 |
| Diagnoses | DG1[].diagnosis_code + diagnosis_desc | A41.9, J18.9 |
| Payer | IN1.payer_type | MEDICARE / COMMERCIAL |
| Plan Name | IN1.plan_name | Medicare Part A |

**Hospital's Status Decision**: Map PV1.patient_class to status
- `I` = Inpatient
- `O` = Observation / Outpatient

---

## 1.2 Parse ORU Labs

Extract lab results. Flag abnormals against rubric thresholds.

| Test | Inpatient Trigger Threshold |
|------|---------------------------|
| Lactate | >2.0 mmol/L |
| WBC | >20 or <2 K/uL |
| Troponin | Above reference range |
| Creatinine | >1.5x baseline or above range |
| Procalcitonin | >0.5 ng/mL |
| BNP | >400 pg/mL |

For each abnormal result, record:
- Test name
- Value
- Reference range
- Flag (H/L/N)
- Whether it meets inpatient trigger threshold

---

## 1.3 Parse ORU Vitals

Extract vital signs over time. Look for:

| Finding | Trigger Criteria |
|---------|-----------------|
| Hypoxia | SpO2 <92% on room air |
| O2 Requirement | >2L needed to maintain SpO2 >92% |
| O2 Escalation | Increasing O2 over time |
| Hypotension | SBP <90 mmHg |
| Tachycardia + Fever | HR >100 AND Temp >101°F |
| Respiratory Distress | RR >24 |

**Track O2 trajectory**: Is it stable, escalating, or de-escalating?

---

## 1.4 Parse MDM Note

Scan the clinical narrative for severity indicators:

| Indicator | Pattern to Find |
|-----------|----------------|
| IV Therapy Required | "IV antibiotics", "IV fluids", "drip", "infusion" |
| Failed Outpatient | "failed oral", "no improvement", "failed outpatient" |
| Severity Score | CURB-65 ≥2, HEART ≥4, APACHE, qSOFA ≥2 |
| Monitoring Required | "continuous", "ICU", "telemetry", "close monitoring" |
| Expected LOS | "admit for X days", "expected stay" |
| Clinical Trajectory | "deteriorating", "worsening", "decompensation risk" |

Extract quoted evidence for each finding.

---

## 1.5 Validate ICD-10 Codes

Use the ICD-10 MCP tools to validate each diagnosis code:

```
For each code in DG1[]:
  1. validate_code(code) → Check is_valid, is_billable
  2. lookup_code(code) → Get full description
  3. If code ends in .9 (unspecified):
     - get_hierarchy(code) → Find more specific options
     - Search MDM for details that could support specificity
  4. Flag issues:
     - Invalid code
     - Non-billable code
     - Unspecified when specifics documented
```

**Output structure**:
```json
{
  "icd10_codes": [
    {
      "code": "A41.9",
      "description": "Sepsis, unspecified organism",
      "valid": true,
      "billable": true,
      "specificity_issue": "Organism could be specified if cultures positive"
    }
  ],
  "issues": ["A41.9 could be more specific if culture results available"]
}
```

---

## 1.6 Assess Severity of Illness

Apply rubric thresholds to clinical data. Document each indicator:

| Category | Finding | Value | Threshold | Met |
|----------|---------|-------|-----------|-----|
| Labs | Lactate | [value] | >2.0 | [Y/N] |
| Labs | WBC | [value] | >20 or <2 | [Y/N] |
| Vitals | SpO2 on RA | [value] | <92% | [Y/N] |
| Vitals | O2 requirement | [value] | >2L | [Y/N] |
| MDM | Severity score | [score] | CURB-65 ≥2 | [Y/N] |

**Severity Summary**: Evaluate overall severity based on:
- Number of indicators met
- Severity of individual findings
- Documented risk scores

---

## 1.7 Assess Intensity of Services

Identify services that require hospital-level care:

| Service | Evidence | Inpatient-Level? |
|---------|----------|------------------|
| IV antibiotics | [from orders/MDM] | Yes |
| Supplemental O2 | [L/min required] | Yes if >2L |
| Continuous monitoring | [telemetry, pulse ox] | Yes |
| IV fluid resuscitation | [volume, protocol] | Yes |
| Frequent nursing assessment | [frequency documented] | Yes if Q2H or more |

**Intensity Summary**: Can these services be safely provided in observation or outpatient setting?

---

## 1.8 Check Two-Midnight Rule Compliance

**Applies to**: Medicare patients only (IN1.payer_type = "MEDICARE")

| Criterion | Finding | Met |
|-----------|---------|-----|
| Expected LOS ≥2 midnights | [documented value] | [Y/N] |
| LOS documented in note | [location in MDM] | [Y/N] |
| Clinical trajectory supports | [evidence] | [Y/N] |

**Compliance Status**: MET / NOT MET / NOT APPLICABLE (non-Medicare)

---

## 1.9 Identify Documentation Gaps

Check for weak or missing documentation:

| Gap Type | Check | Found |
|----------|-------|-------|
| Labs not referenced | Abnormal labs mentioned in MDM? | [Y/N] |
| O2 baseline missing | SpO2 on room air documented? | [Y/N] |
| No clinical reasoning | Admission decision justified? | [Y/N] |
| No failed outpatient | Prior treatment attempt documented? | [Y/N] |
| Severity not documented | Risk score or severity language present? | [Y/N] |
| LOS not stated | Expected length of stay in note? | [Y/N] |

List specific gaps with recommendations.

---

## 1.10 Generate Preliminary Verdict

Based on the assessment, determine:

**Verdict Logic**:
```
IF multiple inpatient triggers met AND no documentation gaps:
  → SUPPORTED (denial risk: LOW)

IF borderline findings OR documentation gaps exist:
  → AT RISK (denial risk: MEDIUM)

IF inpatient triggers absent OR major documentation gaps:
  → LIKELY DENIAL (denial risk: HIGH)
```

**Confidence Level**:
- HIGH: Clear evidence supports verdict
- MEDIUM: Some ambiguity, reviewer judgment needed
- LOW: Significant uncertainty, escalate to physician advisor

---

## 1.11 Write Waypoint File

Save assessment to `waypoints/assessment.json`:

```json
{
  "request_id": "[MRN]_[DATE]_[TIME]",
  "status": "assessment_complete",
  "timestamp": "[ISO timestamp]",

  "patient": {
    "mrn": "[MRN]",
    "age": [age],
    "sex": "[M/F]"
  },

  "admission": {
    "type": "[Inpatient/Observation]",
    "diagnosis": [
      { "code": "[ICD-10]", "description": "[desc]" }
    ],
    "payer": "[payer_type]",
    "payer_name": "[plan_name]",
    "admission_date": "[date]"
  },

  "code_validation": {
    "icd10_codes": [...],
    "issues": [...]
  },

  "medical_necessity": {
    "severity_of_illness": {
      "indicators": [...],
      "summary": "[narrative]"
    },
    "intensity_of_services": {
      "indicators": [...],
      "summary": "[narrative]"
    }
  },

  "two_midnight_rule": {
    "applicable": [true/false],
    "expected_los": "[value]",
    "documented_in_note": [true/false],
    "exceeds_threshold": [true/false],
    "compliance": "[MET/NOT MET/NOT APPLICABLE]"
  },

  "documentation_gaps": [...],

  "preliminary_verdict": "[SUPPORTED/AT RISK/LIKELY DENIAL]",
  "denial_likelihood": "[LOW/MEDIUM/HIGH]",
  "confidence": "[HIGH/MEDIUM/LOW]",
  "rationale": "[explanation]"
}
```

---

## Completion

After writing `waypoints/assessment.json`:

1. Confirm file was written successfully
2. Display summary to user:
   - Patient: [age][sex] | MRN
   - Status: [Inpatient/Observation]
   - Preliminary Verdict: [VERDICT]
   - Denial Risk: [LEVEL]
3. Ask: "Ready to proceed to Step 2: Clinical Review & Letter Generation? (Y/N)"

If user confirms, proceed to `references/02-decision-letter.md`.
