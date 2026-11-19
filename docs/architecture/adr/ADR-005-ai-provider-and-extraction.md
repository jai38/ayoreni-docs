# ADR-005: AI Prescription Extraction Architecture & Provider Strategy

**Date:** 2026-10-12  
**Status:** accepted (Implemented in Phase 2)  
**Deciders:** Principal Architect & AI Integration Engineer  

---

## Context

Prescription document understanding is a core capability of My Health App. Prescriptions feature:
- Diverse clinical formats: Printed hospital slips, handwritten doctor notes, multi-page discharge summaries, government hospital stamps.
- Regional Indian prescription characteristics: Regional doctor handwriting styles, bilingual text (English and Indian languages), common Indian brand names (e.g., Paracetamol, Augmentin, Pantocid, Glycomet, Telma, Rosuvas), generic-brand mixtures, and standard medical abbreviations (`OD` = once daily, `BD`/`BID` = twice daily, `TDS`/`TID` = thrice daily, `QID` = four times daily, `SOS` = as needed, `AC` = before meals, `PC` = after meals, `BBF` = before breakfast).
- Imperfect mobile captures: Skewed camera photos, shadows, crumpled paper, low resolution.
- High medical safety stakes: Hallucinated medicine names, transposed dosages (5mg vs 50mg), or invented durations are unacceptable.
- Privacy & Regulatory compliance: Under the India Digital Personal Data Protection (DPDP) Act and international health privacy standards, sending patient health documents to third-party AI APIs requires contractual certainty regarding data retention, model training exclusions, and processing geography.

We must define an AI integration architecture that maximizes extraction fidelity, enforces strict privacy, prevents hallucinated clinical truth, and remains completely vendor-agnostic.

---

## Decision

### 1. Provider-Independent Abstraction Interface
All AI interaction is decoupled behind a server-side abstraction interface:
```typescript
export interface PrescriptionExtractionProvider {
  /**
   * Extracts structured prescription data from a document buffer.
   */
  extractPrescription(input: {
    documentBuffer: Buffer;
    mimeType: string;
    pageCount: number;
    options?: {
      targetLanguage?: string;
      regionHint?: 'IN' | 'GLOBAL';
    };
  }): Promise<ExtractedPrescriptionPayload>;
}
```

No AI SDKs or API credentials will ever be embedded in the mobile client bundle.

### 2. Multi-Provider Evaluation & Model Selection Policy
We do **not** hardcode Claude 3.5 Sonnet or any other single provider as the permanent production engine. Instead, we define a comparative evaluation matrix across leading multimodal models:
- **Anthropic Claude 3.5 Sonnet**
- **Google Gemini 1.5 Pro / 1.5 Flash**
- **AWS Bedrock / Textract + Claude**
- **OpenAI GPT-4o / GPT-4o-mini**

Before locking in a production provider, the following verification gates must be cleared:
1. **Contractual Privacy & Training Terms:** Verify the provider's exact Terms of Service, Business Associate Agreement (BAA), or Enterprise Data Processing Addendum (DPA). We do **not** claim "zero data retention" without written, contractual verification applicable to our active commercial account.
2. **Regional Processing & Data Residency:** Identify where inference occurs (e.g., whether data stays within India / EU / US) and evaluate compliance with applicable DPDP Act cross-border transfer rules.
3. **Extraction Accuracy on Regional Benchmarks:** Models must be scored against our custom Indian prescription evaluation benchmark (detailed below).
4. **Latency & Operating Cost:** Evaluate per-document cost (e.g. multi-page PDFs) and rate limits under anticipated user load.

### 3. Mock Provider for Development and CI (`MockPrescriptionExtractionProvider`)
To ensure reliable, deterministic, zero-cost, and offline-capable development, we mandate a `MockPrescriptionExtractionProvider`:
- Simulates realistic extraction latency.
- Returns deterministic fixture payloads representing clean prescriptions, ambiguous handwritten prescriptions, missing dates, and multi-page discharge summaries.
- Simulates provider error conditions (rate limit 429, timeout 504, validation failure 422) for automated testing.
- Allows full local development and end-to-end testing without external API credentials or incurring cloud costs.

### 4. Structured Schema Validation with Zod
Every model response must be parsed and validated against strict Zod domain schemas:
```typescript
export const ExtractedMedicineSchema = z.object({
  name_as_written: z.string().min(1),
  normalized_name: z.string().optional(),
  strength: z.string().optional(), // e.g. "500 mg"
  dosage_form: z.enum(["TABLET", "CAPSULE", "SYRUP", "INJECTION", "CREAM", "DROPS", "INHALER", "OTHER"]),
  frequency_text: z.string().optional(), // e.g. "1-0-1", "TDS pc"
  duration_text: z.string().optional(),  // e.g. "5 days"
  instructions: z.string().optional(),
  confidence: z.enum(["HIGH", "UNCERTAIN"]),
  source_page: z.number().int().min(1).default(1),
});

export const ExtractedPrescriptionPayloadSchema = z.object({
  prescription_date: z.string().regex(/^\d{4}-\d{2}-\d{2}$/).optional(),
  date_is_uncertain: z.boolean(),
  doctor_name: z.string().optional(),
  clinic_hospital_name: z.string().optional(),
  documented_diagnosis: z.string().optional(),
  medicines: z.array(ExtractedMedicineSchema),
  unparsed_notes: z.string().optional(),
  validation_warnings: z.array(z.string()),
});
```

Valid JSON alone is insufficient; domain validators reject impossible dates, negative durations, or nonsensical strengths.

### 5. Mandatory Patient Review & Inviolable Truth Boundary
AI extractions are **strictly draft suggestions** stored in `ai_extraction_jobs`. They:
- Never automatically create active medication reminders.
- Never write verified events to the medical timeline.
- Flag ambiguous handwriting or uncertain fields with warning badges.
- Require explicit patient review and confirmation side-by-side with the original image before persistence.

---

### 6. Benchmark Honesty, Provenance & Evaluation Integrity
To ensure rigorous evaluation and avoid deceptive metrics:
1. **Explicit Provenance & Permitted Use:** Every benchmark sample in the evaluation dataset (`modules/ai-extraction/benchmark/dataset.ts`) includes machine-readable provenance metadata:
   - `origin`: Either `SYNTHETIC_DEVELOPER_GENERATED` or `PERMITTED_DEIDENTIFIED`.
   - `license`: Explicit licensing (e.g., `MIT-COMMERCIAL-PERMITTED`, `CC-BY-4.0-DEIDENTIFIED`).
   - `provenance_notes`: Clinical archetype, handwriting characteristics, and verification intent.
   - `permitted_use`: Explicitly authorized for clinical AI extraction benchmarking, training exclusion, and safety validation.
2. **Strict Separation of Contract Testing vs. Empirical Accuracy:**
   - Deterministic test-double verification (e.g. verifying that `MockPrescriptionExtractionProvider` returns expected fixture shapes in CI) is classified strictly as a **harness contract test**, NOT as an empirical evaluation of model accuracy.
   - Mock fixture matching must **never** be cited or reported as evidence of real-world AI extraction capability.
3. **Live AI Integration Privacy & Compliance Gate:**
   - Live third-party AI APIs (Claude, Gemini, OpenAI, Bedrock) remain strictly **disabled** in production until the enterprise Data Processing Addendum (DPA) and Business Associate Agreement (BAA) verifying contractual zero-retention and zero-model-training are legally executed and verified for our specific commercial account.
   - Local and CI test suites run exclusively against offline mock providers and synthetic fixtures, preventing unconsented PHI transmission.
4. **Zero Fabrication of Unmeasured Model Scores:**
   - Model benchmark scorecards must report only empirical data measured on real, audited runs against physical model endpoints under controlled test parameters. No scores may be estimated, extrapolated, or invented.

---

## Prescription Extraction Evaluation Benchmark

A formal evaluation dataset of 32 synthetic/consented test documents is maintained:
1. **Indian Handwritten Prescriptions:** Clinic slips with cursive handwriting and local brand names (e.g., Augmentin 625, Pantocid 40, Dolo 650, Calpol).
2. **Standard Abbreviations:** Regimens written in regional formats (`1-0-1`, `0-1-0`, `TDS pc`, `SOS`, `HS`).
3. **Multipage Hospital Discharge Summaries:** 3-5 page clinical summaries containing past medical history, in-hospital vitals, and discharge medication tables.
4. **Degraded & Skewed Mobile Captures:** Phone photographs with low lighting, folded creases, and doctor stamps overlapping text.
5. **Incomplete Documents:** Prescriptions with missing dates, missing frequencies, or missing doctor names (verifying that missing fields remain `null` or marked `date_is_uncertain: true` and are **never** fabricated).
6. **Negative Controls:** Lab invoices, billing receipts, or unrelated documents (verifying graceful rejection with `validation_warnings`).

---

## Consequences

### Positive
- Production provider can be selected, replaced, or upgraded dynamically via configuration.
- Local development and automated testing run without third-party API dependencies or costs.
- Hallucinations cannot compromise patient clinical history.
- Compliance posture is evidence-backed rather than based on unverified marketing claims.
- Benchmark results are honest, transparent, and legally defensible with full fixture provenance.

### Negative
- Requires maintaining the `MockPrescriptionExtractionProvider` and benchmarking harness alongside provider implementations.

