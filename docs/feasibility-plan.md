# Smart Nasal Swab — Feasibility Plan

> Research-backed feasibility memo for an AI-driven nasal-swab diagnostic platform
> that identifies pathogens, quantifies mucosal antibodies, infers stage of
> infection, and recommends treatment with a clinician in the loop.
>
> Method: 6 parallel research sub-agents (clinical, competitive, ML/LLM,
> regulatory, hardware, CDS). Full reports under `/research/`.

---

## 1. The idea
A self-collected nasal swab + AI/LLM platform that:
1. **Identifies the pathogen** (multiplex respiratory panel — COVID, Flu A/B, RSV, etc.).
2. **Quantifies mucosal antibody titer** (IgA primarily, IgG secondary) to infer where
   the patient is in the disease course (early / peak / convalescent).
3. **Recommends treatment** tailored to pathogen, stage, host immune state, and
   patient context — with a real clinician reviewing every Rx.

ML decision trees encode the guidelines. An LLM (Claude Opus 4.7) glues the pieces,
explains the rationale, runs the dialogue, and is **never** the primary diagnostician
or autonomous prescriber.

---

## 2. Verdict

**Overall plausibility: 6/10 today, 7–8/10 by 2029** — *only if the v1 product is
narrowly scoped*. The full vision (pathogen + quantitative antibody + stage +
treatment Rx, all from one swab, autonomously) is **not** feasible in 2026. A
defensible MVP that captures the unique wedge **is**.

| Sub-claim | 2026 plausibility | Notes |
|---|---|---|
| Multiplex pathogen ID from self-collected nasal swab | 9/10 | Solved. Visby, Aptitude Metrix, BIOFIRE SPOTFIRE shipping. |
| 3–4-pathogen panel at home | 8/10 | Aptitude Metrix EUA Feb 2025 is the template. |
| **Quantitative mucosal IgA from same swab at POC** | 3/10 today, 6/10 by 2029 | Single biggest blocker — nobody has solved sample-volume / mucin / circadian normalization. |
| Stage-of-infection inference (early/peak/convalescent) | 3/10 today, 5/10 by 2029 | Biology is real (viral load + IgA/IgG + ISG signature); ±2–3 day error bars at best. |
| Treatment **recommendation** (clinician-in-loop) | 7/10 | Doable as Class II SaMD; LLM-as-glue not diagnostician. |
| Treatment **autopilot Rx** (no clinician) | 1/10 | Regulatory + liability cliff. Don't ship this. |

### Does it already exist?
No. No single product combines pathogen ID + mucosal antibody titer + AI-driven
treatment recommendation. Category leaders cover one slice each:
- **Visby / Aptitude / BIOFIRE** = multiplex pathogen ID, no AI, no antibody.
- **Adaptive T-Detect / Karius** = immune state, but blood-based and lab-based.
- **OpenEvidence / Glass / Hippocratic** = clinical AI, but no test integration.

**The three-in-one swab is genuine white space.**

### The graveyard
Cue Health (Ch. 7, May 2024 — FDA revoked EUA over device modifications).
Lucira/Pfizer (killed March 2025, EUAs revoked Oct 2025).
Ellume (liquidated 2023 after Class I recall of 2.2M tests).
Forward Health ($400M shutdown Nov 2024 — AI-replaces-doctor without reimbursement).
Theranos (fabricated tech).

**Patterns that kill companies in this space:**
- Single-pathogen products die when pandemic demand fades.
- Hardware QA failure is existential.
- Consumer subscription on physical kits has brutal unit economics.
- "AI replaces doctor" without insurance reimbursement burns out at $400M+.
- Insurance billing creativity → federal investigation (uBiome).

---

## 3. Recommended product (MVP, <$2M, 18 months)

**One-line pitch:** *A 3-in-1 nasal-swab cassette that tells you what virus you have,
how far into it you are, and what to do — with a real clinician in the loop, every
time.*

### Hardware
- **Dual-strip LFA cassette + smartphone camera + cardboard hood.** No custom reader
  in v1. (Cue's $249 reader + $65 cartridge is the cautionary tale.)
- **Strip 1 — pathogen multiplex:** OEM/private-label COVID + Flu A/B + RSV antigen
  LFA from Healgen (FDA De Novo Sept 2024) or SD Biosensor.
- **Strip 2 — mucosal IgA ladder (the differentiated piece):** custom 3-line LFA
  developed with a contract LFA mfr (Lumos, Abingdon, BBI, DCN). 9–12 months,
  ~$300–600k NRE. Lines saturate at calibrated capture-Ab concentrations so the AI
  can bin titer into 3–4 levels.
- **Phone-camera reader:** kinetic capture at t = 5 / 10 / 15 min through a printed
  fiducial + color-calibration patch on the cassette. The app is the differentiator,
  not the hardware.
- **COGS target:** <$3/test. **Retail:** $20–30 cash-pay. **Reimbursement = upside,
  not the plan.**

### Software architecture
See [`architecture.md`](./architecture.md) for the diagram. In summary:

1. **On-device** (TFLite, <50MB): cassette detection, glare/blur guard, kinetic frame
   capture.
2. **Cloud inference plane**:
   - ResNet-50 strip-line regressor (per analyte; line intensity + T/C ratio).
   - **Bayesian Neural-ODE** stage-of-infection head — mechanistic backbone
     (Baccam/Perelson target-cell + adaptive-immune ODE) with learned rate constants.
     Outputs a posterior over days-since-onset with calibrated credible intervals.
     **This is the technical moat.**
   - Cross-attention fusion transformer (BiomedCLIP-init) over
     {strip embeddings, IgA ladder, vitals, symptoms NLP, optional EHR}.
3. **Deterministic rules layer** ← *source of truth*. IDSA/CDC/AAP/NICE encoded as
   FHIR PlanDefinition + CQL. Closed candidate-set of guideline-permitted regimens.
   DDI engine (FDB/Lexicomp/RxNorm + DDInter) prunes the set. **Safe to ship even
   without the LLM.**
4. **LLM orchestrator** (Claude Opus 4.7, low-temp, JSON-schema enforced). Cannot
   introduce a drug not in the candidate-set. Citation-required RAG over
   IDSA/CDC/UpToDate. Llama Guard 3 input/output filter. Haiku critic on every draft.
5. **Clinician-review gate** via CDS Hooks Card / SMART-on-FHIR app. Confidence +
   uncertainty band, inline citations, override-with-reason. **No one-click
   prescribe** — Rx flows through normal EHR order entry.

**Non-negotiable invariants:**
- Rules layer is the source of truth. LLM is glue + explainer + dialogue.
- Every recommendation is reproducible from {rule-set version, FHIR snapshot, model
  version, prompt hash}.
- Abstain by default when inputs are stale or rules and LLM disagree.
- PCCP envelope drafted into the v1 FDA submission so retraining doesn't trigger a
  new 510(k).

---

## 4. Regulatory path

**The CDS carve-out (§3060(a) of the Cures Act) is NOT available.** The AI processes
IVD signal, which fails criterion #1. Plan around being a regulated device.

| Layer | Pathway | Notes |
|---|---|---|
| IVD assay | **De Novo + PCCP** | No predicate covers OTC quantitative mucosal antibody. MDUFA V fee: $40,527 (small biz). **Apply for FDA Breakthrough Device designation up front** — 60-day decision, free, unlocks Sprint discussions. |
| AI/ML SaMD layer | **Class II 510(k) with PCCP** | Under FDA Dec 2024 PCCP final guidance + Jan 2025 Lifecycle Management draft. Build subgroup performance + FAVES disclosure into v1 (HTI-1 makes this de facto required). |
| CLIA | OTC at-home doesn't trigger CLIA | But state ELR (Electronic Lab Reporting) for flu/COVID/RSV/pertussis is mandatory — build day one. |
| HIPAA + HBNR | Hybrid-entity structure | FTC expanded HBNR to cover health apps (Apr 2024). California AB 2013 (Jan 2026) + Colorado AI Act (Feb 2026) require training-data and risk-mgmt disclosures. |
| EU | IVDR Class C | Notified Body + EU Reference Lab. EU AI Act high-risk (Article 6). Defer until US authorization. |

**Liability posture:** ship as decision *support*. Clinician is prescriber-of-record.
Carry product liability + tech E&O ($25M+ aggregate). 14% YoY rise in AI-CDS claims
2022–2024; insurers now ask on renewals.

---

## 5. 24-month roadmap

| Quarter | Milestone | Spend |
|---|---|---|
| Q1–Q2 2026 | Incorporate; raise $1.5–2M seed; engage IVD regulatory counsel (Hogan Lovells / Goodwin / Ropes & Gray); file FDA Q-Sub + Breakthrough Device request. | $300k |
| Q2–Q3 2026 | Custom mucosal-IgA ladder strip development with contract LFA mfr; OEM agreement with Healgen/SD Biosensor for antigen multiplex; phone-app + ResNet-50 model trained on Davies/Frontiers/StyleGAN2 corpus. | $600k |
| Q3 2026 | Launch **RUO version** under 21 CFR 809.10(c). No clinical claims. Use it to build paired strip-image dataset under IRB at 3–4 academic medical centers, draft PCCP envelope, collect longitudinal IgA + Ct trajectories for Bayesian-ODE training. | $200k |
| Q4 2026 – Q2 2027 | Pivotal multi-site clinical study (≥1,500 paired specimens, lay-user). IEC 62366-1 usability for OTC. Subgroup analyses pre-registered (sex/age/race/ethnicity/comorbidity per FDA June 2024 guidance). | $400k |
| Q3 2027 | **De Novo + PCCP submission** as combination IVD + SaMD. Use Breakthrough Sprint to compress review. | $100k FDA fee + counsel |
| Q1 2028 | Authorization; soft launch in 2–3 states; ELR live; CDS Hooks Card live in pilot Epic sites; layer in payor-billed CLIA-lab variant via partner lab. | — |

Total runway: **$1.5–2M seed → $5–8M Series A** at ~Q2 2027. Pattern-match: Karius
($100M Series C, Khosla/5AM/Gilde-led), Visby ($135M+ Series E, BARDA), SiPhox ($32M,
Khosla/Intel).

---

## 6. 5-year roadmap

- **Years 3–4:** First 510(k) line-extensions under the PCCP — add Mycoplasma, Group A
  strep, host-response RNA signature (à la Inflammatix). Begin EU IVDR Class C
  Notified Body engagement (BSI / TÜV SÜD); align with EU AI Act high-risk obligations
  before Aug 2027 deadline.
- **Year 5:** Gen-2 device with PMA upgrade where treatment recommendation is the
  labeled use — gain Riegel-style preemption against state tort claims and command
  premium reimbursement. Add reusable reader if cartridge economics close.

---

## 7. Critical risks + kill criteria

| Risk | Kill criterion |
|---|---|
| Mucosal-IgA quantification can't be standardized for sample volume/mucin/circadian | If IgA-ladder CV >40% across 200 healthy volunteers in pilot, kill the antibody-stage claim and ship pathogen-only. |
| FDA pushes treatment-recommendation feature into Class III / PMA | Drop to "guideline information display" (avoid single-directive language) and resubmit Class II. |
| Cue/Lucira-style consumer-subscription unit economics | Cap retail ≤$30 cash; never depend on subscription; lean into Test-to-Treat (Walmart/CVS/Amazon Clinic) channel. |
| LLM hallucination harms patient | Rule-engine override + dual-LLM critic + clinician gate. If post-market surveillance shows >0.5% LLM-caused override events in 90 days, freeze LLM layer to deterministic templates. |
| HIPAA / HBNR breach | Hybrid-entity structure, BAAs day one, PHI never leaves cloud inference plane to LLM unredacted. |

---

## 8. "Could it do all of it, do everything?"

**Honest answer: no, not in v1, and shipping the autonomous-Rx version would kill the
company.**

But the *narrowed* product — pathogen ID + crude IgA-titer bin + stage hint +
clinician-reviewed treatment suggestion — is genuinely novel, defensible, and fits a
$1.5–2M seed. The wedge is the **IgA-ladder strip + Bayesian Neural-ODE stage
estimator**. Everything else is integration.

If the founder already operates a HIPAA-aware telehealth stack (clinician dashboard,
audit log, RBAC, Stripe seats, FHIR plumbing, agent orchestrator), that collapses
6–9 months of build time. **This is the rare case where "I already have the rails"
turns a $10M+ company into a $2M-MVP company.**

---

## Research index
Full sub-agent reports with citations (PubMed IDs, FDA guidance PDFs, arXiv, etc.):
- [`research/clinical.md`](../research/clinical.md) — what's biologically measurable
- [`research/competitive.md`](../research/competitive.md) — landscape + white-space
- [`research/ml-llm.md`](../research/ml-llm.md) — model architecture
- [`research/regulatory.md`](../research/regulatory.md) — FDA/CLIA/HIPAA/HTI-1
- [`research/hardware.md`](../research/hardware.md) — biosensor + reader stack
- [`research/cds.md`](../research/cds.md) — treatment recommendation + safety
