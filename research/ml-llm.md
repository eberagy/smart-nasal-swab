# ML / Decision-Tree / LLM Architecture Memo

Scope: model stack only. Hardware/wet-lab assumed solved. Opinionated; defended
against 2024–2026 literature.

## 1. Signal-Interpretation Layer
One model per transducer, all reduced to a common 256-d embedding.

- **Smartphone strip image:** Two-stage — YOLOv8-n cassette detector → **ResNet-50
  line-intensity regressor** (fallback EfficientNet-B0). MDPI Biosensors 2025 shows
  ResNet-50 wins (R²=0.94, MSE=0.003) and beats ViT-B/16 at <10⁵-image scale.
  Reference designs: SMARTAI-LFA (Nat Comm 2023, 98% across 135 clinical tests) and
  Davies et al. Cell Reports Med 2022 (sensitivity 92→97.6%). Use **StyleGAN2-ADA**
  augmentation (Frontiers AI 2023).
- **qPCR/LAMP curve:** **1D-Transformer** with linear projection + positional
  embeddings — exactly Moniri's T-CDAN (93% curve / 97% sample accuracy on
  carbapenem-resistant clinical isolates). Domain-adversarial alignment is mandatory
  for reagent-lot drift.
- **Electrochemical I-V:** **Shallow 1D-CNN + XGBoost** on engineered peak features.
  GBT wins under 5k samples.
- **SERS spectra:** **1D-CNN with multi-branch adaptive attention** (J. Hazardous
  Materials 2025, 98.6% species / 99.5% AMR). Pretrain self-supervised with **BYOL**
  on unlabeled spectra. Add adversarial cross-device standardization head
  (Spectrochim. Acta 2025) — non-negotiable.

## 2. Antibody-Titer Regression
Calibrated regression on T/C ratio + line intensity + kinetics. ResNet-50 with two
heads: (a) categorical {neg/weak/pos/void}, (b) continuous log-titer (BAU/mL) with
**Gaussian NLL loss** for aleatoric uncertainty. SCAISY (MDPI 2023) gets within ~30%
of ELISA. If 3+ frames captured during develop window, add a **GRU over per-frame
embeddings** (Nat Comm 2024). Datasets: Davies UK COMPARE LFA, Frontiers/StyleGAN2
COVID corpus, NIST RDT photo library, partner-licensed Roche/Bosch pilots, NHANES for
population priors. Floor: 5–10k labeled images per pathogen + 1k titrated
dilution-series, augmented to 50k via StyleGAN2-ADA / DSAWGAN.

## 3. Stage-of-Infection Inference — defensible moat
**Hierarchical Bayesian neural ODE.** Underlying biology *is* a coupled ODE
(Baccam/Perelson target-cell-limited + adaptive-immune model); IgA seroconverts
d6–15, IgG peaks d16–30. Neural ODE parameterizes unknown rate constants while
preserving the mechanistic backbone — extrapolates where pure ML fails
(immunocompromised). Hierarchical Bayesian layer (NumPyro/Stan) gives **calibrated
credible intervals** on days-since-onset — clinically defensible. Inputs: viral load
(Ct), IgA/IgG ratio, symptom-day, vitals. Output: posterior over (days-since-onset,
peak-load-day, infectious-window-end). MoE/Cox/pure-Transformer all rejected.

## 4. Decision-Tree / Clinical-Rules Layer
Rules win where auditability or hard safety dominates:
- **Hard contraindications, dosing, drug-drug, pregnancy, renal:** CDS Hooks + CQL
  over FHIR + SNOMED/RxNorm/LOINC. JAMIA Open 2025 validates this stack.
- **Guideline-grade triage** (IDSA antiviral eligibility, Centor/McIsaac, oseltamivir
  <48h): **Drools / OpenCDS** (40k+ US facilities).
- **Reportable disease flags:** pure rules.

ML owns pathogen call, titer, stage, trajectory. Rules own *what to do*. Rule layer
is a **safety override** — if ML recommends X but a rule forbids it (allergy, AKI,
age), the rule wins and the LLM is told why.

## 5. LLM Orchestration Layer
LLM is **not the diagnostician** — it's glue, explainer, and dialogue.

- Tool-calling to invoke signal models, Bayesian stage estimator, rule engine; emits
  structured `Recommendation` JSON.
- **RAG over IDSA/CDC/UpToDate/Sanford** with citation-required decoding. npj Digital
  Medicine 2025 shows safety-framed RAG hits 1.47% hallucination; Wada drove
  hallucinations 8% → 0%.
- Multi-turn dialogue — AMIE matched/beat PCPs on diagnostic accuracy + empathy in
  OSCE simulations; MedGemma 4B beats Med-Gemini at smaller size on VQA.
- **Not** primary diagnosis. CRAFT-MD (Johri, Nat Med 2024) shows SOTA LLMs degrade
  sharply in multi-turn history-taking vs. single-shot vignettes.
- **Hallucination control:** forced JSON schemas, citation-required RAG,
  self-reflection pass, RxNorm-constrained decoding, two-model critic (Haiku audits
  Opus drafts), display ML-derived confidence (never LLM-narrated).

## 6. Multi-Modal Fusion
**Intermediate (cross-attention) fusion with bottleneck fusion tokens** — BiomedCLIP
/ Med-Gemini / MedFuse-TransNet style. Early fusion breaks under modality dropout;
late fusion discards cross-modal correlation. Each modality (strip, amp, SERS,
vitals, PubMedBERT-symptom, EHR foundation model, optional PRS) → 256-d embedding →
4-layer Transformer with 8 fusion tokens cross-attending all modalities → multi-label
pathogen + titer regression + Bayesian-ODE heads. BiomedCLIP weights initialize
image/text branches.

## 7. Training-Data Realities
- Strip CNN: 5–20k per pathogen line (Davies UK COMPARE, Frontiers StyleGAN2 corpus,
  Kaggle, partner pilots, synthetic).
- Amp-curve Transformer: 30–50k curves; FIND/WHO/BARDA panels + simulated; T-CDAN-
  style domain adaptation mandatory.
- SERS: 5–10k spectra/pathogen; SSL-pretrain on unlabeled bacterial-isolate corpora.
- Bayesian ODE: longitudinal serial Ct + serology + symptom-diary cohorts
  (HEROES/RECOVER, UK Biobank long-COVID, partners). Few thousand trajectories;
  prior does heavy lifting.
- Context: MIMIC-IV, eICU-CRD, NHANES, All of Us v8, partner EHR; FDA MAUDE for
  adverse-event surveillance.
- LLM RAG corpus: IDSA, CDC, WHO, AAP, UpToDate (license), Sanford, NICE.

## 8. Validation & Monitoring
**Calibration:** ECE + Brier + reliability diagrams per pathogen *and* per subgroup.
Post-hoc temperature scaling + isotonic regression. **Drift:** dual monitor —
covariate-side MMD/BBSD on embeddings (Soin Nat Comm 2024) + output-side
adaptive-windowing on PPV/NPV vs. PCR truth (PMC 8627243). Trigger PCCP-bounded
retrain on threshold breach. **FDA pathway:** 510(k) + PCCP under FDA Jan 2025 final
guidance + Dec 2024 PCCP guidance. PCCP enumerates change scope, retraining
methodology, re-validation protocol, rollback. Multi-site prospective trial powered
for PPA/NPA per pathogen vs. PCR (mirror Visby's 96.2/96.9/93.2% PPA targets) plus
subgroup non-inferiority. Pre-register endpoints; pin model versions in audit trail.

## 9. Recommended Stack

```
              ┌────────── PATIENT + DEVICE ──────────┐
              │ Strip · Amp · SERS · I-V             │
              │ Symptoms · Vitals · EHR · (PRS opt.) │
              └──────────────────────────────────────┘
                              │
        ┌─────── ON-DEVICE (TFLite, <50MB) ────────┐
        │ YOLOv8-n cassette · EfficientNet-B0      │
        │ pre-screen · blur/glare guard            │
        └─────────────────┬────────────────────────┘
                          ▼ (encrypted upload)
   ┌──── CLOUD INFERENCE PLANE ─────────────────────────┐
   │ Per-modality encoders → 256-d embeddings           │
   │  ResNet-50 strip · 1D-Transformer (T-CDAN) amp     │
   │  1D-CNN+attn SERS · XGBoost+1D-CNN I-V             │
   │  PubMedBERT symptoms · EHR-FM history              │
   │       │                                            │
   │       ▼                                            │
   │  CROSS-ATTN FUSION TRANSFORMER (BiomedCLIP-init)   │
   │   • Multi-label pathogen (Sigmoid + Platt)         │
   │   • Titer regression (Gaussian NLL)                │
   │   • Bayesian Neural-ODE stage head                 │
   │       │                                            │
   │       ▼                                            │
   │  RULE / CDS LAYER (Drools + OpenCDS + CDS-Hooks)   │
   │   SNOMED · RxNorm · LOINC · IDSA/CDC rules         │
   │   Hard safety overrides                            │
   │       │                                            │
   │       ▼                                            │
   │  LLM ORCHESTRATOR (Claude Opus 4.7)                │
   │   Tool-calling · RAG IDSA/CDC/UpToDate             │
   │   Cite-required · self-reflect · Haiku critic      │
   │   Patient + clinician dialogue                     │
   │       │                                            │
   │       ▼                                            │
   │  MONITORING (ECE · MMD drift · adaptive PPV/NPV    │
   │   audit log · PCCP-bounded retraining)             │
   └────────────────────────────────────────────────────┘
                          │
                          ▼
        Clinician dashboard · Patient app · FHIR write-back
```

**Split rule:** anything PHI/RAG/Bayesian-ODE → cloud. Anything sub-second user
feedback (cassette alignment, retake) → on-device. The LLM never sees raw signal —
only structured outputs from ML and rule engine.

## Sources
1. [Davies et al., Cell Reports Medicine 2022](https://www.cell.com/cell-reports-medicine/fulltext/S2666-3791(22)00339-1)
2. [SMARTAI-LFA, Nature Communications 2023](https://www.nature.com/articles/s41467-023-38104-5)
3. [Frontiers AI 2023 — COVID LFT + StyleGAN2](https://www.frontiersin.org/journals/artificial-intelligence/articles/10.3389/frai.2023.1235204/full)
4. [MDPI Biosensors 2025 — ML LFA quantification](https://www.mdpi.com/2079-6374/15/1/19)
5. [SCAISY, MDPI Biosensors 2023](https://www.mdpi.com/2079-6374/13/6/623)
6. [Moniri et al. — T-CDAN amp-curve transformer](https://pubmed.ncbi.nlm.nih.gov/37028376/)
7. [J. Hazardous Materials 2025 — SERS adaptive-attention CNN](https://www.sciencedirect.com/science/article/abs/pii/S0304389425016103)
8. [Spectrochim. Acta 2025 — SERS cross-device standardization](https://www.sciencedirect.com/science/article/pii/S1386142525012387)
9. [Med-Gemini, arXiv 2404.18416](https://arxiv.org/html/2404.18416v2)
10. [MedGemma Technical Report, arXiv 2507.05201](https://arxiv.org/html/2507.05201v1)
11. [AMIE / Nature Medicine 2024](https://www.nature.com/articles/s41591-024-03328-5)
12. [CRAFT-MD, Nat Med 2024](https://pubmed.ncbi.nlm.nih.gov/39747685/)
13. [npj Digital Medicine 2025 — clinical-safety LLM hallucination framework](https://www.nature.com/articles/s41746-025-01670-7)
14. [FDA PCCP Final Guidance (Dec 2024 / Jan 2025)](https://www.fda.gov/regulatory-information/search-fda-guidance-documents/marketing-submission-recommendations-predetermined-change-control-plan-artificial-intelligence)
15. [JAMIA Open 2025 — CDS-Hooks + CQL + FHIR](https://academic.oup.com/jamiaopen/article/8/4/ooaf085/8219442)
16. [Visby Medical Respiratory Test — FDA 510(k) + CLIA, Feb 2025](https://www.prnewswire.com/news-releases/visby-medical-receives-fda-clearance-and-clia-waiver-for-point-of-care-respiratory-health-test-302385373.html)
17. [Nat Comm 2024 — empirical drift detection in medical imaging](https://www.nature.com/articles/s41467-024-46142-w)
