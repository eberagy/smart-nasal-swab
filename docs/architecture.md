# Smart Nasal Swab — Architecture

> Detailed architecture diagram for the v1 product. See
> [`feasibility-plan.md`](./feasibility-plan.md) for the strategic context.

## Data flow

```
              ┌────────── PATIENT + DEVICE ──────────┐
              │ Nasal swab → dual-strip LFA cassette │
              │  Strip 1: COVID/Flu A/B/RSV antigen  │
              │  Strip 2: Mucosal-IgA ladder (3-line)│
              │ Phone camera + cardboard hood        │
              │ + symptoms, vitals, optional EHR     │
              └──────────────────────────────────────┘
                              │
        ┌─────── ON-DEVICE (TFLite, <50 MB) ────────┐
        │ YOLOv8-n cassette detector                │
        │ EfficientNet-B0 quality pre-screen        │
        │ Glare / blur / fiducial-alignment guard   │
        │ Kinetic capture at t = 5 / 10 / 15 min    │
        └─────────────────┬─────────────────────────┘
                          ▼ (encrypted upload, FHIR R4)
   ┌──── CLOUD INFERENCE PLANE ─────────────────────────┐
   │ Per-modality encoders → 256-d embeddings           │
   │  ResNet-50 strip line regressor (T/C ratio)        │
   │  1D-Transformer for amp curves (T-CDAN style)      │
   │  PubMedBERT for symptom NLP                        │
   │  EHR-FM for prior history (optional)               │
   │      │                                             │
   │      ▼                                             │
   │ CROSS-ATTN FUSION TRANSFORMER (BiomedCLIP-init)    │
   │  ├── Multi-label pathogen head (Sigmoid + Platt)   │
   │  ├── IgA-titer regression (Gaussian NLL)           │
   │  └── Bayesian Neural-ODE stage head ← MOAT         │
   │      • Mechanistic backbone:                       │
   │        Baccam/Perelson target-cell ODE             │
   │        + adaptive-immune (IgA/IgG) ODE             │
   │      • Learned rate constants (NumPyro/Stan)       │
   │      • Outputs: posterior over days-since-onset,   │
   │        peak-load-day, infectious-window-end        │
   │      │                                             │
   │      ▼                                             │
   │ DETERMINISTIC RULES LAYER  ←  source of truth      │
   │  • IDSA / CDC / AAP / NICE as FHIR PlanDefinition  │
   │    + CQL                                           │
   │  • SNOMED / RxNorm / LOINC                         │
   │  • Hard safety overrides:                          │
   │      pregnancy contraindications, peds dosing,     │
   │      renal/hepatic adjustment, allergies, DDI      │
   │  • DDI engine: FDB / Lexicomp / RxNorm + DDInter   │
   │  • Output: closed candidate-set of guideline-      │
   │    permitted regimens (or "supportive only" /      │
   │    "refer to clinician")                           │
   │      │                                             │
   │      ▼                                             │
   │ LLM ORCHESTRATOR (Claude Opus 4.7)                 │
   │  • Low temp, JSON-schema enforced                  │
   │  • Cannot introduce a drug not in candidate-set    │
   │  • Citation-required RAG over                      │
   │    IDSA / CDC / UpToDate / Sanford                 │
   │  • Llama Guard 3 input/output filter               │
   │  • Haiku critic on every draft                     │
   │  • Patient + clinician dialogue                    │
   │      │                                             │
   │      ▼                                             │
   │ MONITORING                                         │
   │  • ECE + Brier + reliability per pathogen          │
   │  • MMD / BBSD covariate-drift on embeddings        │
   │  • PPV / NPV adaptive-windowing vs. PCR truth      │
   │  • PCCP-bounded retraining triggers                │
   │  • HTI-1 source-attribute panel + audit log        │
   └────────────────────────────────────────────────────┘
                          │
                          ▼
        CDS Hooks Card / SMART-on-FHIR app
                          │
                          ▼
   Clinician dashboard · Patient app · TeleMD video visit
   (clinician is prescriber-of-record; Rx flows through
    normal EHR order entry — NEVER one-click prescribe)
                          │
                          ▼
   FHIR write-back · State ELR (flu/COVID/RSV/pertussis)
   audit log · outcomes telemetry → PCCP retraining loop
```

## Split rule
- **On-device** (sub-second user feedback): cassette alignment, retake guidance, frame
  quality. TFLite, no PHI leaves the phone in this stage.
- **Cloud** (PHI-handling, requires audit): all per-modality encoders, fusion
  transformer, Bayesian Neural-ODE, rules layer, LLM, RAG.
- **LLM never sees raw signal** — only structured outputs from the ML models and the
  rules engine.

## Model inventory + training data sources

| Model | Family | Training data |
|---|---|---|
| Cassette detector | YOLOv8-n | Synthetic + 2k labeled cassette photos |
| Quality pre-screen | EfficientNet-B0 | Same |
| Strip line regressor | ResNet-50, two heads (categorical + Gaussian-NLL log-titer) | Davies UK COMPARE LFA, Frontiers/StyleGAN2 COVID corpus, NIST RDT photo library, partner pilots, NHANES priors. ~5–20k images per pathogen line, augmented to 50k via StyleGAN2-ADA. |
| Amp-curve interpreter | 1D-Transformer (T-CDAN) | 30–50k curves; FIND/WHO/BARDA panels + simulated. Domain-adversarial alignment for reagent-lot drift. |
| Stage estimator | Bayesian Neural-ODE (NumPyro/Stan over learned rate constants) | Longitudinal serial Ct + serology + symptom-diary cohorts (HEROES/RECOVER, UK Biobank long-COVID). Few thousand trajectories; mechanistic prior does heavy lifting. |
| Fusion transformer | Cross-attention, BiomedCLIP-init | Paired multi-modal samples from RUO pilot |
| LLM | Claude Opus 4.7 | Zero fine-tune. RAG corpus: IDSA, CDC, WHO, AAP, UpToDate (license), Sanford, NICE. |
| Critic | Claude Haiku 4.5 | Same |
| Guardrails | Llama Guard 3 (off-the-shelf) | — |

## Validation
- **Pre-market:** multi-site prospective trial powered for PPA/NPA per pathogen vs.
  reference PCR (mirror Visby's 96.2/96.9/93.2% targets), plus subgroup non-inferiority
  per FDA June 2024 Diverse Populations guidance.
- **Post-market:** ECE + Brier per pathogen *and* per subgroup; covariate drift
  (MMD/BBSD on embeddings — Soin et al. Nat Comm 2024); output drift (adaptive PPV/NPV
  windowing). Threshold breach → PCCP-bounded retrain.
- **Reproducibility:** every recommendation reconstructable from {rule-set version,
  FHIR snapshot, model version, prompt hash}.
