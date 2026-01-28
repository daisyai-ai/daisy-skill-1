# Tool Integration

## ICD-10 Code Tools

Use these tools to validate and enrich diagnosis codes from the ADT message.

### Validate Diagnosis Codes

For each code in DG1 segment:

```
ICD10_Codes:validate_code
  code: "A41.9"
```

Check response for:
- `is_valid`: Confirms code exists
- `is_billable`: Can be used on claims
- Flag non-billable codes as documentation gaps

### Lookup Code Details

Get full context for a diagnosis:

```
ICD10_Codes:lookup_code
  code: "A41.9"
```

Returns description, category, and clinical context.

### Check Code Hierarchy

Understand severity positioning:

```
ICD10_Codes:get_hierarchy
  code: "A41.9"
```

Use hierarchy to:
- Identify if more specific codes exist
- Understand severity relative to parent/sibling codes
- Flag "unspecified" (.9) codes when specifics documented in MDM

### Search for Better Codes

If MDM contains specific details not reflected in DG1:

```
ICD10_Codes:search_codes
  query: "sepsis due to staphylococcus"
  code_type: "diagnosis"
```

Flag as documentation gap if specific code exists but generic used.

## CMS Coverage Tools (Optional)

For Medicare patients, lookup coverage policies:

### Check National Coverage

```
CMS_Coverage:search_national_coverage
  keyword: "sepsis"
  document_type: "ncd"
```

### Check Local Coverage

```
CMS_Coverage:search_local_coverage
  keyword: "observation"
  document_type: "lcd"
```

Use coverage policies to:
- Validate medical necessity language
- Identify specific documentation requirements
- Support two-midnight rule interpretation

## Tool Usage Patterns

### Pattern 1: Code Validation Loop

```
For each diagnosis in ADT.DG1:
  1. validate_code → confirm billable
  2. lookup_code → get description
  3. Compare description to MDM language
  4. Flag discrepancies
```

### Pattern 2: Specificity Check

```
If code ends in .9 (unspecified):
  1. get_hierarchy → find sibling codes
  2. Search MDM for specific details
  3. If details present → flag documentation gap
  4. Suggest more specific code
```

### Pattern 3: Severity Validation

```
For high-severity diagnoses (sepsis, MI, stroke):
  1. lookup_code → confirm severity
  2. Cross-reference with labs/vitals
  3. Verify MDM documents severity markers
  4. Strong inpatient signal if all align
```
