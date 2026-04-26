# Smart Nasal Swab

> *A 3-in-1 nasal-swab cassette that tells you what virus you have, how far into it
> you are, and what to do — with a real clinician in the loop, every time.*

AI-driven point-of-care diagnostic platform combining (1) multiplex respiratory
pathogen ID, (2) mucosal IgA titer quantification for stage-of-infection inference,
and (3) clinician-reviewed treatment recommendation.

---

## Verdict (TL;DR)

| | 2026 | 2029 |
|---|---|---|
| **Overall plausibility** | 6/10 *(narrowly scoped MVP)* | 7–8/10 |
| Multiplex pathogen ID at home | 9/10 (solved) | — |
| Quantitative mucosal IgA at POC | 3/10 (the moat) | 6/10 |
| Stage-of-infection inference | 3/10 (±2–3d error bars) | 5/10 |
| Treatment rec. *with clinician* | 7/10 | — |
| Autopilot Rx (no clinician) | 1/10 — **don't ship this** | — |

**Does it already exist?** No. Visby/Aptitude/BIOFIRE = pathogen multiplex only.
Adaptive T-Detect / Karius = immune state but blood + lab. OpenEvidence/Glass =
clinical AI but no test integration. **The three-in-one swab is genuine white
space.**

The single biggest blocker is mucosal-IgA standardization at POC (sample volume,
mucin, time-of-day variance). The technical moat — the **IgA-ladder strip + Bayesian
Neural-ODE stage estimator** — is exactly the unsolved-but-tractable problem.

---

## Recommended MVP (<$2M, 18 months)

**Hardware:** Dual-strip LFA cassette + smartphone camera + cardboard hood. No
custom reader. Strip 1 = OEM/private-label COVID + Flu A/B + RSV antigen multiplex
(Healgen / SD Biosensor). Strip 2 = custom mucosal-IgA ladder (3 calibrated lines,
contract LFA shop, ~$300–600k NRE). COGS <$3, retail $20–30 cash-pay.

**Software:** ResNet-50 strip-line regressor → Bayesian Neural-ODE stage head →
deterministic IDSA/CDC/AAP rules layer (FHIR PlanDefinition + CQL) → Claude Opus 4.7
LLM orchestrator constrained to the rule-engine candidate-set → CDS Hooks Card with
clinician sign-off. **LLM is glue, not diagnostician.**

See [`docs/architecture.md`](./docs/architecture.md) for the full diagram.

---

## 24-month roadmap

| Quarter | Milestone |
|---|---|
| Q1–Q2 2026 | Incorporate; raise $1.5–2M seed; FDA Q-Sub + Breakthrough Device request |
| Q2–Q3 2026 | Custom IgA-ladder strip dev; OEM agreement; phone app + ResNet-50 trained |
| Q3 2026 | Launch RUO version; build IRB-cleared paired dataset at 3–4 AMCs |
| Q4 2026 – Q2 2027 | Pivotal multi-site clinical study (≥1,500 paired specimens) |
| Q3 2027 | **De Novo + PCCP submission** (combination IVD + SaMD) |
| Q1 2028 | Authorization; soft launch in 2–3 states; ELR live |

Total runway: $1.5–2M seed → $5–8M Series A at ~Q2 2027. Pattern-match: Karius
($100M Series C, Khosla/5AM/Gilde), Visby ($135M+ Series E, BARDA), SiPhox ($32M,
Khosla/Intel).

---

## Repo layout

```
docs/
  feasibility-plan.md   ← the synthesized memo (start here)
  architecture.md       ← system diagram + model inventory
research/
  clinical.md           ← what's biologically measurable from a nasal swab
  competitive.md        ← landscape, white-space, the graveyard
  ml-llm.md             ← model architecture (signal, stage, fusion, LLM)
  regulatory.md         ← FDA/CLIA/HIPAA/HTI-1/EU IVDR pathway
  hardware.md           ← biosensor + reader + cartridge stack
  cds.md                ← treatment recommendation + safety + liability
```

---

## Status

Pre-incorporation. This repo currently contains research and planning artifacts only
— no code yet. Next concrete steps live in
[`docs/feasibility-plan.md` §4–§5](./docs/feasibility-plan.md).
