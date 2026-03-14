# Insurance Program Template Pack

This repository now includes a reusable **template form** and a **generation prompt template** for producing synthetic insurance-program records with built-in validation flags.

## 1) Insurance Program Form Template

Use this as a standardized intake/output form (CSV-compatible):

```csv
Member ID,Full Name,Date of Birth,Gender,Policy Number,Program Type,Coverage Start Date,Coverage End Date,Primary Diagnosis,Secondary Diagnosis,Requested Service,Provider Name,Estimated Cost (USD),Pre-existing Conditions,Current Medications,Allergies,Approval Status,Rejection/Exception Reason,Is Valid,Issue
M001,Jane Doe,1988-04-12,F,POL-100245,Comprehensive,2026-01-01,2026-12-31,Type 2 Diabetes,Hypertension,Endocrinology Consultation,Dr. Smith,250,Diabetes Type 2,Metformin,None,Approved,,True,
M002,John Roe,1974-09-30,M,POL-100246,Chronic Care,2026-01-15,2026-12-31,Asthma,,Pulmonary Function Test,City Pulmonary Clinic,180,Asthma,Albuterol,Aspirin,Approved,,True,
M003,Ana Kim,1992-11-21,F,POL-100247,Comprehensive,2026-02-01,2027-01-31,Bacterial Infection,,Penicillin Course,Metro Hospital,95,None,None,Penicillin,Rejected,Medication conflicts with documented allergy,False,Prescribed Penicillin despite Penicillin allergy
```

---

## 2) Prompt Template for Synthetic Insurance Data

You can copy this template into your LLM workflow and fill placeholders.

```text
You are a helpful assistant designed to generate data.

Generate {{ROW_COUNT}} rows of synthetic insurance program records in valid CSV format only.

Use these columns exactly:
- Member ID
- Date of Birth
- Gender (M/F)
- Policy Number
- Program Type
- Coverage Start Date
- Coverage End Date
- Medical History
- Current Medications
- Allergies
- Lab Results (Glucose mg/dL)
- Diagnosis
- Treatment/Service Plan
- Approval Status
- Is Valid (True/False)
- Issue

Rules:
1. Member ID format: M followed by a three-digit number (e.g., M006, M319).
2. Intentionally include some invalid rows.
3. For invalid rows, set Is Valid=False and explain clearly in Issue.
4. Include realistic error types such as:
   - Allergy contradictions (covered drug conflicts with allergy)
   - Medical history vs plan mismatch (e.g., diabetic member with no diabetes-related treatment)
   - Lab result vs diagnosis mismatch
   - Other plausible errors (e.g., impossible DOB, coverage dates reversed, incompatible approval status)
5. Keep valid rows clinically and administratively consistent.
6. Return CSV only (no markdown).
```

---

## 3) Python Wrapper Template

```python
def generate_insurance_data(client, model, row_count=100):
    messages = [
        {
            "role": "user",
            "content": f"""
You are a helpful assistant designed to generate data.
Generate {row_count} rows of synthetic insurance program records in valid CSV format only.

Columns:
Member ID,Date of Birth,Gender,Policy Number,Program Type,Coverage Start Date,Coverage End Date,Medical History,Current Medications,Allergies,Lab Results (Glucose mg/dL),Diagnosis,Treatment/Service Plan,Approval Status,Is Valid,Issue

Rules:
- Member ID format: M + three digits.
- Include both valid and intentionally invalid rows.
- If Is Valid is False, Issue must explain the specific inconsistency.
- Include realistic inconsistencies (allergy contradiction, diagnosis/lab mismatch, date inconsistencies, etc.).
- Return CSV only.
"""
        }
    ]

    response = client.chat.completions.create(
        model=model,
        messages=messages
    )

    return response.choices[0].message.content.replace("```csv", "").replace("```", "")
```
