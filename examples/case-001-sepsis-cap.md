# Example Output: Case 001 - Sepsis/CAP

**Input**: sample-data/ (ADT, labs, vitals, MDM for 78M with sepsis/pneumonia)

---

## Step 1 Output: waypoints/assessment.json

```json
{
  "request_id": "MRN-998877_20231027_143000",
  "status": "assessment_complete",
  "timestamp": "2023-10-27T14:45:00Z",

  "patient": {
    "mrn": "MRN-998877",
    "age": 78,
    "sex": "M"
  },

  "admission": {
    "type": "Inpatient",
    "diagnosis": [
      { "code": "A41.9", "description": "Sepsis, unspecified organism" },
      { "code": "J18.9", "description": "Pneumonia, unspecified organism" }
    ],
    "payer": "MEDICARE",
    "payer_name": "Medicare Part A",
    "admission_date": "2023-10-27"
  },

  "code_validation": {
    "icd10_codes": [
      { "code": "A41.9", "description": "Sepsis, unspecified organism", "valid": true, "billable": true },
      { "code": "J18.9", "description": "Pneumonia, unspecified organism", "valid": true, "billable": true }
    ],
    "issues": [
      "A41.9 could be more specific if culture results available",
      "J18.9 could be J18.1 (lobar) based on CXR findings"
    ]
  },

  "medical_necessity": {
    "severity_of_illness": {
      "indicators": [
        { "finding": "Lactate elevated", "value": "3.1 mmol/L", "threshold": ">2.0", "met": true },
        { "finding": "WBC elevated", "value": "18.4 K/uL", "threshold": ">11.0", "met": true },
        { "finding": "Hypoxia on arrival", "value": "SpO2 88%", "threshold": "<92%", "met": true },
        { "finding": "O2 escalation", "value": "RA→2L→4L NC", "threshold": ">2L required", "met": true },
        { "finding": "Tachycardia + fever", "value": "HR 102, 101.4°F", "threshold": "HR>100 + Temp>101", "met": true },
        { "finding": "CURB-65 score", "value": "3", "threshold": "≥2", "met": true }
      ],
      "summary": "Multiple severity indicators present. Patient demonstrates sepsis physiology with respiratory compromise requiring escalating oxygen support."
    },
    "intensity_of_services": {
      "indicators": [
        { "service": "IV antibiotics", "evidence": "Rocephin + Azithromycin IV ordered", "inpatient_level": true },
        { "service": "Oxygen therapy", "evidence": "4L NC required to maintain SpO2 >92%", "inpatient_level": true },
        { "service": "IV fluid resuscitation", "evidence": "30cc/kg sepsis bolus initiated", "inpatient_level": true },
        { "service": "Continuous monitoring", "evidence": "Sepsis protocol, pulse ox monitoring", "inpatient_level": true }
      ],
      "summary": "Services require hospital-level care. IV antibiotics, oxygen therapy, and sepsis protocol cannot be safely delivered in observation or outpatient settings."
    }
  },

  "two_midnight_rule": {
    "applicable": true,
    "expected_los": "3-5 days",
    "documented_in_note": true,
    "exceeds_threshold": true,
    "compliance": "MET"
  },

  "documentation_gaps": [],

  "preliminary_verdict": "SUPPORTED",
  "denial_likelihood": "LOW",
  "confidence": "HIGH",
  "rationale": "Clinical documentation demonstrates severity of illness and intensity of services appropriate for inpatient-level care. Multiple sepsis criteria met with escalating oxygen requirements. Two-midnight threshold exceeded with expected LOS of 3-5 days documented."
}
```

---

## Checkpoint Display

```
╔═══════════════════════════════════════════════════════════════════╗
║                    CLINICAL REVIEW CHECKPOINT                      ║
╠═══════════════════════════════════════════════════════════════════╣
║  Patient: 78M | MRN-998877                                         ║
║  Current Status: Inpatient                                         ║
║  Payer: Medicare Part A                                            ║
╚═══════════════════════════════════════════════════════════════════╝

AI ASSESSMENT: SUPPORTED (High Confidence)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

The AI found sufficient clinical evidence supporting inpatient status.

SEVERITY OF ILLNESS
• Sepsis with lactate 3.1 mmol/L (elevated)
• WBC 18.4 K/uL (elevated)
• Hypoxia requiring 4L O2 (oxygen dependent)
• Tachycardia (HR 102) with fever (101.4°F)
• CURB-65 score: 3 (14% mortality risk)

INTENSITY OF SERVICES
• IV antibiotics required (Rocephin, Azithromycin)
• Supplemental oxygen (4L NC)
• IV fluid resuscitation (sepsis bolus)
• Continuous pulse ox monitoring

TWO-MIDNIGHT COMPLIANCE
• Expected LOS: 3-5 days ✓
• Documented in MDM: Yes ✓
• Exceeds 2-midnight threshold: Yes ✓

CODE VALIDATION
• A41.9 (Sepsis) - Valid, billable
• J18.9 (Pneumonia) - Valid, billable

DOCUMENTATION GAPS: None identified

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Based on this data, the payer is UNLIKELY to deny this claim.

Do you agree with this assessment? (Agree/Disagree): Agree
```

---

## Step 2 Output: outputs/status-validation-report.md

# Status Validation Report

**Patient:** 78M | MRN: MRN-998877
**Admission Date:** October 27, 2023
**Current Status:** Inpatient
**Payer:** Medicare Part A

---

## Screening Result

**Verdict:** SUPPORTED
**Denial Risk:** LOW

Your inpatient status decision is well-supported. Based on the clinical documentation, the payer is unlikely to deny this claim.

---

## ICD-10 Code Validation

| Code | Description | Valid | Billable | Issue |
|------|-------------|-------|----------|-------|
| A41.9 | Sepsis, unspecified | ✓ | ✓ | None |
| J18.9 | Pneumonia, unspecified | ✓ | ✓ | None |

**Code Status:** All codes valid and billable.

*Recommendation:* Consider organism-specific sepsis code (e.g., A41.51) when cultures return for optimal specificity. Consider J18.1 (lobar pneumonia) if CXR confirms consolidation pattern.

---

## Medical Necessity Assessment

### Severity of Illness

The patient demonstrates multiple clinical indicators of acute illness requiring inpatient-level monitoring and intervention:

| Indicator | Value | Threshold | Status |
|-----------|-------|-----------|--------|
| Lactate | 3.1 mmol/L | >2.0 | Elevated |
| WBC | 18.4 K/uL | >11.0 | Elevated |
| SpO2 on room air | 88% | <92% | Abnormal |
| O2 requirement | 4L NC | >2L | Elevated |
| Temperature | 101.4°F | >101°F | Febrile |
| Heart rate | 102 bpm | >100 | Tachycardic |
| CURB-65 Score | 3 | ≥2 | High Risk |

**Summary:** Severity of illness supports hospital-level care. The combination of sepsis physiology (elevated lactate, WBC) and respiratory compromise (hypoxia requiring escalating O2) indicates acute illness beyond outpatient capacity.

### Intensity of Services

The patient requires services that can only be safely delivered in a hospital setting:

| Service | Evidence | Inpatient-Level |
|---------|----------|-----------------|
| IV antibiotics | Rocephin + Azithromycin IV | ✓ |
| Oxygen therapy | 4L NC to maintain SpO2 >92% | ✓ |
| IV fluid resuscitation | 30cc/kg sepsis bolus | ✓ |
| Continuous monitoring | Sepsis protocol, pulse ox | ✓ |

**Summary:** Intensity of services appropriate for inpatient status. These interventions require nursing oversight and monitoring that cannot be replicated in observation or outpatient settings.

---

## Two-Midnight Rule Compliance

For Medicare Part A, inpatient status requires expected length of stay spanning two or more midnights.

| Criterion | Finding | Met |
|-----------|---------|-----|
| Expected LOS ≥2 midnights | "3-5 days" documented | ✓ |
| LOS stated in physician note | MDM Plan section | ✓ |
| Clinical trajectory supports | Sepsis + CAP + O2 need | ✓ |

**Compliance:** MET — Documentation supports stay exceeding two-midnight threshold.

---

## Documentation Checklist

- [x] Baseline SpO2 documented (88% on room air)
- [x] Lab abnormalities referenced in assessment
- [x] Severity score documented (CURB-65 = 3)
- [x] Failed outpatient therapy documented ("oral Amox x 2 days")
- [x] Expected LOS explicitly stated
- [x] Medical decision-making explains clinical reasoning

**No documentation gaps identified.**

---

## Clinical Reviewer Attestation

| Field | Value |
|-------|-------|
| Reviewer Agreement | AGREE |
| Reviewer Notes | — |
| Review Date | 2023-10-27T15:00:00Z |

---

## Action Items

**Required:** None — proceed to claim submission.

**Optional Enhancements:**
1. Update A41.9 to organism-specific code when cultures finalize
2. Specify J18.1 (lobar pneumonia) if CXR confirms consolidation pattern

---

*AI-assisted screening tool | Human clinical review required | 2023-10-27T15:00:00Z*
