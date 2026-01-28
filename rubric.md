# Status Determination Rubric

**The LLM evaluates the admission packet against these criteria.**

---

## Verdict Options

| Verdict | Meaning |
|---------|---------|
| **SUPPORTED** | Evidence justifies hospital's status decision |
| **AT RISK** | Borderline - may be denied, documentation gaps |
| **LIKELY DENIAL** | Evidence does not support the status |

---

## Inpatient Triggers (Any ONE = Strong Inpatient Signal)

### From Labs (oru_labs.json)

| Finding | Threshold | Why It Matters |
|---------|-----------|----------------|
| Lactate elevated | > 2.0 mmol/L | Sepsis/hypoperfusion |
| Troponin elevated | Above ref range | Cardiac injury |
| WBC critical | > 20 or < 2 | Severe infection or immunocompromise |
| Creatinine jump | > 1.5x baseline | Acute kidney injury |

### From Vitals (oru_vitals.json)

| Finding | Threshold | Why It Matters |
|---------|-----------|----------------|
| O2 required | > 2L to maintain SpO2 > 92% | Respiratory dependency |
| O2 escalating | Increasing O2 over time | Deterioration |
| Hypotension | SBP < 90 | Hemodynamic instability |
| Tachycardia + fever | HR > 100 + Temp > 101 | SIRS criteria |

### From MDM Note (mdm_note.json)

| Finding | Look For | Why It Matters |
|---------|----------|----------------|
| IV therapy required | "IV antibiotics", "IV fluids", "drip" | Not outpatient-feasible |
| Failed outpatient | "failed oral", "no improvement at home" | Escalation needed |
| High severity score | CURB-65 ≥ 2, HEART score ≥ 4, etc. | Documented risk |
| Monitoring required | "continuous monitoring", "ICU", "telemetry" | Intensity of service |

---

## Observation Signals (Absence of Inpatient Triggers)

| Finding | Suggests Observation |
|---------|---------------------|
| Labs normal or mildly abnormal | Low severity |
| Vitals stable, no O2 needed | Low intensity |
| Oral medications sufficient | Outpatient-feasible |
| "Rule out" without positive findings | Diagnostic workup only |
| Expected resolution < 24 hours | Short stay |

---

## Two-Midnight Rule (Medicare Only)

If payer = MEDICARE:
- Expected LOS ≥ 2 midnights → Supports Inpatient
- Expected LOS < 2 midnights → Likely Observation

**Look for LOS clues in MDM:** "admit for 3-5 days", "anticipated discharge in 48 hours"

---

## Documentation Gap Flags

Flag if evidence exists but documentation is weak:

| Gap | Example |
|-----|---------|
| Labs abnormal but not mentioned in MDM | Lactate 3.1 not referenced in note |
| O2 given but no SpO2 documented | "On 4L NC" but no baseline SpO2 |
| "Inpatient" without severity language | Note just says "admit" with no clinical reasoning |
| No failed outpatient documented | Jumped straight to IV without trial of oral |

---

## How to Apply

1. **Scan labs** for any inpatient triggers
2. **Scan vitals** for instability or O2 dependency
3. **Read MDM** for severity scores, IV therapy, failed outpatient
4. **Check payer** - if Medicare, apply two-midnight lens
5. **Compare** evidence to hospital's status decision
6. **Output** verdict + specific reasons from the data
