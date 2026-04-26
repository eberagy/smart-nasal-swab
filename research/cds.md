# Treatment Recommendation / CDS Layer Memo (April 2026)

Scope: the layer that, given a pathogen call + inferred infection stage from a
nasal swab, emits a treatment recommendation. This is the most regulated, most
safety-critical surface in the product.

---

## 1. Applicable clinical guidelines

| Pathogen | Anchor guideline | Tx posture |
|---|---|---|
| Influenza A/B | IDSA 2018; CDC Antivirals Summary | Outpatient oseltamivir 5d or single-dose baloxavir within 48h. Recommended for high-risk; "consider" for healthy adults if <48h. Pregnant → oseltamivir. **Recommendable as a suggestion**, but Rx requires clinician. |
| SARS-CoV-2 | NIH/IDSA COVID-19 Tx Guidelines; CDC Outpatient Care | Nirmatrelvir-r within 5d for high-risk; remdesivir 3-day outpatient or molnupiravir as alternates. **DDI screening (CYP3A4) mandatory** — no auto-recommendation without med list. |
| RSV (peds) | AAP 2025 RSV Prevention Statement; CDC | Treatment is supportive; nirsevimab/clesrovimab are prevention. Ribavirin only narrow immunocompromised. **Default supportive + red-flag triage; never auto-recommend ribavirin.** |
| Rhinovirus | None | Supportive only. **Ideal stewardship surface.** |
| Group A strep | IDSA 2012; CDC | Penicillin/amoxicillin × 10d; cephalexin/clinda/azithro if PCN-allergic. Diagnosis still requires Centor + RADT/culture → **not a swab-only Rx decision**. |
| Mycoplasma | IDSA/PIDS CAP; 2024 MRMP consensus | Azithromycin first-line; doxy >8y or FQ (adults) if macrolide-resistant. Rising MRMP → **clinician-only**. |
| Bordetella | CDC; AAP Red Book | Macrolide; reportable → **must trigger clinician + reporting**. |
| Acute bacterial sinusitis | IDSA 2012; AAP 2013 | Amox-clav first-line if criteria met; watchful waiting only mild + reliable f/u. **Diagnosis is clinical, not swab-based — flag, don't prescribe.** |

**Bright line:** swab confirms a pathogen, but most guidelines condition therapy on
severity, comorbidity, pregnancy, age, time-since-onset. Tool should **suggest**,
never autonomously prescribe.

---

## 2. Antimicrobial stewardship — where AI actually helps

Highest-leverage move is **recommending NOT to prescribe**. ~30% of US outpatient
abx Rx are unnecessary (CDC). For viral URI/rhinovirus/flu/RSV/SARS-CoV-2, the swab
kills the "maybe bacterial" hedge driving over-prescription.

Real evidence:
- Stanford personalized-antibiogram ML: cut vanc + pip-tazo use ~69% while
  preserving coverage (Comms Medicine 2022, basis for current Stanford ASP work).
- 2025 systematic review/meta-analysis of AI-CDS in ASP: AI tools modestly
  outperform traditional risk-scoring on resistance prediction; behavior-change
  effect is workflow-dependent.
- Sanford Guide + Stewardship Assist (Stanford and others): vendor-reported lift in
  narrow-spectrum adherence; no RCT-grade public outcomes.
- FirstLine: institution-customizable mobile guidelines; outcome literature thin.

A swab + LLM that says "viral — no antibiotic, here's the supportive script" is a
defensible product wedge.

> ⚠ Note: research agent could not verify a Stanford "MERIDIAN" antimicrobial-
> stewardship study. Closest matches are the Stanford personalized-antibiogram work
> (Comms Medicine 2022) and a 2025 *Stenotrophomonas maltophilia* RCT (Implementation
> Science). Treat MERIDIAN as unverified until a citation is produced.

---

## 3. CDS Hooks / SMART on FHIR

- **CDS Hooks 2.0**: service fires on `patient-view`, `order-sign`, or custom
  `medication-prescribe`; returns Cards (info / suggestion / app-link) the clinician
  can accept.
- **SMART on FHIR app**: launched from Card; pulls allergies, meds, problems,
  pregnancy, weight, renal function via FHIR R4.
- **Production status:** CDS Hooks GA in Epic (all sites), limited deployment in
  Oracle Health/Cerner; Cerner sunset DSTU2, target R4. Epic surfaces Cards in BPAs
  and sidebar; in-basket Cards now first-class.
- **Auth:** SMART App Launch 2.0 with `launch/patient`, `user/Observation.read`,
  `patient/MedicationRequest.read`, OAuth2 dynamic client registration where
  supported.

Clean pattern: device pushes result → internal API → CDS Hooks `patient-view` Card
with LLM-drafted recommendation + "Review evidence" SMART app launch.

---

## 4. LLM-as-CDS — measured performance

Diagnosis-with-data:
- **GPT-4 base:** 86.1% MedQA-USMLE.
- **Med-PaLM 2:** 86.5% MedQA, +19% over Med-PaLM (Nature Medicine 2024).
- **Med-Gemini-L 1.0:** SOTA 10/14 medical benchmarks; +13.2% top-10 NEJM-CPC over
  AMIE (arXiv 2404.18416).
- **AMIE:** top-10 dx accuracy 59.1% vs 33.6% unassisted clinicians (p=0.04);
  clinicians + AMIE > clinicians alone.
- **MAI-DxO (Microsoft):** 304 NEJM-CPC sequential cases — MAI-DxO + o3 hits 80%
  (max-config 85.5%) vs 20% mean for 21 practicing physicians; -20% diagnostic cost
  vs physicians (arXiv 2506.22405, 2025).
- **Open Medical-LLM Leaderboard** (HuggingFace): standardized comparison across
  MedQA/MedMCQA/PubMedQA/MMLU-clinical.

Treatment-selection: gap narrows. LLMs match/beat physicians on USMLE-style
multi-choice tx items but degrade on medication-safety with patient context — *npj
Digital Medicine* 2025 scoping review and *Frontiers in Pharmacology* case-eval
found LLMs miss real DDIs (e.g., metoprolol+verapamil) at clinically meaningful
rates. **LLMs are better diagnosticians than prescribers.** This is exactly why
prescribing needs deterministic rails.

---

## 5. Hallucination / safety patterns

Layered approach:
1. **Structured outputs** (JSON schema / function calling) — pathogen, severity
   bucket, age band, pregnancy, candidate drugs from a *closed set*. Free text only
   for patient-facing rationale.
2. **Deterministic rule fallback / RAG over guideline corpus** — IDSA/CDC/AAP as
   FHIR PlanDefinition + CQL or hand-built decision tables. LLM proposes; rules
   arbitrate.
3. **Llama Guard 3** (input/output classification) + **NeMo Guardrails**
   (programmable rails, RAG fact-check, self-consistency hallucination check).
   NVIDIA medical-RAG pattern is the most concrete public reference; arXiv
   2409.17190 "Enhancing Guardrails for Safe and Secure Healthcare AI."
4. **Dual-LLM generate-then-critique** — drafter + different-family critic at lower
   temp; disagreement → human escalation. Pharmacovigilance Guardrails (Sci Reports
   2025; arXiv 2407.18322) implements this.
5. **"I don't know" calibration** — explicit refusal training + abstention threshold
   tied to logprobs / verifier disagreement. AMIE / Med-Gemini cite calibrated
   abstention as a key safety lever.

---

## 6. FDA + legal posture on LLM treatment text

- **FDA CDS Final Guidance (Sept 28, 2022):** the four §520(o)(1)(E) criteria.
  **Criterion 3: a *single specific* treatment directive (vs. a list of options the
  HCP independently reviews) is generally a device** because it invites automation
  bias. Per-patient signal-derived outputs (a swab-derived therapy pick) push toward
  device status.
- **AI/ML SaMD Action Plan** finalized Dec 2024. Companions: June 2024 *Transparency
  for ML-Enabled Devices*; Dec 2024 *PCCP Final Guidance*; Jan 2025 *Lifecycle
  Management & Marketing Submission* draft.
- **As of April 2026 the FDA has not authorized a generative-AI / LLM SaMD.** Nov
  2024 Digital Health Advisory Committee flagged genAI as the next frontier; FDA
  signaled it will tag devices that contain foundation models.
- **ONC HTI-1** (Jan 9, 2024; DSI criterion live Jan 1, 2025): 31 source attributes
  for Predictive DSIs in certified health IT — intended use, training data, bias
  mitigation, validation, ongoing maintenance. Certified-EHR partners will demand
  HTI-1 attestations.

**Where the line falls:**
- "Patient appears to have flu A. CDC/IDSA recommend oseltamivir 75 mg BID × 5d for
  high-risk within 48h; review renal/pregnancy." → *information / list of options*
  → non-device-ish.
- "Prescribe oseltamivir 75 mg BID × 5d" with one-click sign → *single directive* →
  device, regulated.
- The swab itself is an IVD; building Rx logic on top amplifies regulatory exposure.

---

## 7. Clinician-in-the-loop UX

Patterns now expected by HTI-1, Epic AI Trust & Assurance, and AMA augmented-
intelligence principles:
- Confidence + uncertainty band, numerically.
- Inline citations to IDSA/CDC/AAP/UpToDate per recommendation line, clickable to
  source.
- Show what changed the recommendation (pregnancy, renal, allergy, peds dose) with
  the FHIR resource that triggered each modifier.
- One-click override with structured reason capture (audit-grade).
- **No single-click accept on Rx.** Force a guideline review screen.
- Audit log: viewer, action, FHIR snapshot, model+version, prompt hash. HTI-1
  source-attribute panel one click away.
- Non-interruptive surfacing (sidebar Card > modal BPA) — Epic 2025 guidance
  explicitly flags alert fatigue.

---

## 8. Failure modes and how to catch them

| Failure | Catch |
|---|---|
| Antibiotic over-Rx for viral URI | Hard rule: viral pathogen + no superinfection markers → *suppress* abx options, surface supportive script. |
| Bacterial superinfection on top of viral | Severity rule (fever curve, duration >10d, biphasic course) → escalate to clinician. |
| DDI (esp. Paxlovid + CYP3A4) | Deterministic engine over `MedicationStatement` (FDB/Lexicomp/RxNorm + DDInter), *not* LLM. LLM summarizes, never adjudicates. |
| Pregnancy contraindication | Hard gate on pregnancy `Condition` + LMP; block doxy/FQ/ribavirin; force clinician on any antiviral. |
| Pediatric dosing | mg/kg with weight-required gate; reject if weight stale >30d or absent. *Pediatric Research* 2025 shows LLMs compute peds doses correctly given clean inputs but should not be the source of truth — use Lexicomp Peds / Sanford Peds. |
| Reportable disease (pertussis, etc.) | Pathogen-keyed rule auto-attaches public-health reporting card. |

**Architecture answer: rule engine on top of the LLM, not a fine-tuned guardrail
model.** Rules are auditable, freezeable, and survive FDA review; a guardrail LLM is
opaque and drifts. Use guardrail models only for NL safety filtering (toxicity,
jailbreak, PHI leak).

---

## 9. Liability landscape (2024–2026)

- **No published US case law yet** holding an AI-CDS vendor liable for a wrong
  treatment recommendation as of Q1 2026. *Dickson v. Dexcom* (LA 2024) is the
  closest medical-device precedent and concerns FDA preemption.
- **Standard of care is shifting.** May 2024 ALI Restatement of Medical Malpractice
  moves from strict customary practice to "reasonable care," creating dual-direction
  liability — harm from using flawed AI *and* harm from not using AI where it has
  become standard (Milbank Quarterly 2025; Chambers Healthcare AI 2025).
- **Claims data:** 2025 reporting cites a 14% rise in AI-tool-related malpractice
  claims 2022→2024, mostly diagnostic AI in radiology/cardiology/oncology.
- **Physician remains the legal prescriber.** Blind acceptance does not insulate
  the clinician; vendor product-liability / negligent-design is the open frontier.
- **Malpractice insurers** (The Doctors Co., Coverys 2025 advisories) now ask on
  renewals which AI tools are in use and demand documented human review.
- **FTC** has signaled (2024–2025 health-AI enforcement actions in the
  GoodRx/Cerebral lineage) that overstated AI accuracy claims are deceptive-practice
  exposure.

Implication: ship as decision *support*, log every override, keep clinician as
prescriber-of-record, carry product liability + tech E&O, disclose performance
honestly per HTI-1.

---

## 10. Recommended architecture

```
SWAB DEVICE (IVD)
  pathogen + viral load + inferred stage
        │ FHIR Observation (LOINC-coded)
        ▼
CONTEXT FETCH (SMART on FHIR R4)
  demo, weight, pregnancy, allergies, MedicationStatement,
  Condition (CKD, immunosuppression), recent Encounter
        ▼
(i) DETERMINISTIC RULES LAYER  ← ground truth
  IDSA / CDC / AAP / NICE encoded as FHIR PlanDefinition + CQL.
  Inputs: pathogen, age, pregnancy, renal, weight, allergies.
  Outputs: candidate-set of guideline-permitted regimens
  (or "supportive only" / "refer"). DDI engine prunes set.
  This layer alone must be safe to ship.
        ▼
(ii) LLM FRAMING LAYER (Claude / GPT-4o-class)
  Input: candidate-set + structured patient context.
  Output: JSON {ranked_options[], rationale,
    patient_explanation, citations[], confidence, abstain?}.
  Constraints: low temp, schema enforced, citations resolve,
  drugs MUST be in candidate-set.
  Guardrails: NeMo Guardrails (RAG fact-check, self-consistency)
  + Llama Guard 3. Dual-LLM critic checks schema, citation
  integrity, drug-in-candidate-set.
        ▼
(iii) CLINICIAN-REVIEW GATE (CDS Hooks Card / SMART app)
  Surfaces ranked options, citations, modifiers, confidence,
  override-with-reason. NEVER one-click prescribe — Rx flows
  through normal EHR order entry.
  Logs: model version, prompt hash, FHIR snapshot, rule-set
  version, decision, latency. HTI-1 source-attribute panel
  one click away.
        ▼
EHR audit log + outcomes telemetry (feeds HTI-1 ongoing maintenance)
```

Invariants:
- **Rules layer is the source of truth.** LLM cannot introduce a drug rules did not
  whitelist.
- **No autonomous Rx.** Clinician signs in EHR's normal order pathway.
- **Reproducibility:** every recommendation reconstructable from {rule-set version,
  FHIR snapshot, model version, prompt hash}.
- **Abstain by default** when inputs are stale (weight, pregnancy unknown for
  child-bearing-age) or when rule layer + LLM disagree.
- **Pre-market posture:** position v1 as non-device CDS by hewing to the four
  §520(o)(1)(E) criteria. Plan SaMD De Novo when v2 adds any single-directive or
  autonomous behavior.

---

## Sources

- [FDA, Clinical Decision Support Software Final Guidance (Sept 28, 2022)](https://www.federalregister.gov/documents/2022/09/28/2022-20993/clinical-decision-support-software-guidance-for-industry-and-food-and-drug-administration-staff)
- [FDA, Artificial Intelligence in Software as a Medical Device](https://www.fda.gov/medical-devices/software-medical-device-samd/artificial-intelligence-software-medical-device)
- [ONC HTI-1 Final Rule (healthit.gov)](https://www.healthit.gov/topic/laws-regulation-and-policy/health-data-technology-and-interoperability-certification-program)
- [Mintz, "HTI-1 Final Rule Introduces New Transparency Requirements for AI" (2024)](https://www.mintz.com/insights-center/viewpoints/2146/2024-01-08-hhs-onc-hti-1-final-rule-introduces-new-transparency)
- [IDSA 2018 Seasonal Influenza Guideline](https://www.idsociety.org/practice-guideline/influenza/)
- [CDC, Influenza Antivirals Summary for Clinicians](https://www.cdc.gov/flu/hcp/antivirals/summary-clinicians.html)
- [CDC, COVID-19 Outpatient Treatment Clinical Care](https://www.cdc.gov/covid/hcp/clinical-care/outpatient-treatment.html)
- [IDSA 2012 Group A Streptococcal Pharyngitis Guideline](https://www.idsociety.org/practice-guideline/streptococcal-pharyngitis/)
- [IDSA Acute Bacterial Rhinosinusitis Guideline](https://www.idsociety.org/practice-guideline/rhinosinusitis/)
- [AAP/CDC, RSV Immunization Guidance for Infants](https://www.cdc.gov/rsv/hcp/vaccine-clinical-guidance/infants-young-children.html)
- [HL7 CDS Hooks 2.0](https://cds-hooks.hl7.org/)
- [Med-Gemini, "Capabilities of Gemini Models in Medicine" (arXiv 2404.18416)](https://arxiv.org/abs/2404.18416)
- [Singhal et al., "Toward expert-level medical question answering with large language models" — Med-PaLM 2 (Nature Medicine 2024)](https://www.nature.com/articles/s41591-024-03423-7)
- [Microsoft Research, "Sequential Diagnosis with Language Models" — MAI-DxO (arXiv 2506.22405)](https://arxiv.org/abs/2506.22405)
- [NVIDIA, Medical RAG with NeMo Guardrails](https://developer.nvidia.com/blog/develop-secure-reliable-medical-apps-with-rag-and-nvidia-nemo-guardrails/)
- ["Need for Guardrails with LLMs in Medical Safety-Critical Settings" (Sci Reports 2025)](https://www.nature.com/articles/s41598-025-09138-0)
- [Milbank Quarterly, "AI and Liability in Medicine" (2025)](https://www.milbank.org/quarterly/articles/artificial-intelligence-and-liability-in-medicine-balancing-safety-and-innovation/)
