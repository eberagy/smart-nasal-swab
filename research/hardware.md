# Hardware / Biosensor Stack Memo

Scope: physical device only. The AI layer is assumed; this memo is about what
hardware can plausibly produce signals an AI can interpret to (a) detect pathogens,
(b) quantify mucosal antibodies, (c) infer stage of infection.

## 1. Sample collection

**Site tradeoffs.** Nasopharyngeal (NP) is the gold standard at ~98% sensitivity vs
PCR but is uncomfortable, requires a trained collector, and is a non-starter for
at-home use. Anterior nares (AN) self-collection sits at 82–88% sensitivity vs NP —
a 12–18% drop driven mostly by viral-load-dependent missed detections (nasal swabs
miss low-viral-load patients). Mid-turbinate (MT) performs comparably to AN. For a
consumer device, AN is the only realistic site.

**Self-swab reliability (FDA EUA data).** Untrained-user studies show very high
concordance with trained collection: Lucira CHECK-IT 100% agreement on contrived
samples and 91.1% PPA / 93% in symptomatics vs RT-PCR. BinaxNOW Self Test: 84.6%
PPA, 98.5% NPA. Cue: 97.4% PPA, 100% NPA. Pattern: molecular self-tests reach
near-lab sensitivity; antigen self-tests trade ~10–15 sensitivity points for cost
and speed.

**Swab + buffer.** Industry standard is nylon-flocked (Copan FLOQSwab) — >90%
elution efficiency, no internal core. Foam (polyurethane) is sometimes preferred for
direct-to-strip antigen tests because it releases mucus better in low-volume buffer;
flocked dominates molecular workflows. Standard elution is 0.3–1.0 mL of a Tris- or
PBS-based proprietary buffer with surfactant + protease-inhibitor.

## 2. Detection chemistries — pathogen

| Chemistry | LOD | Time | Multiplex | $/test | POC-ready? | Quantitative? |
|---|---|---|---|---|---|---|
| Au-NP lateral flow antigen | ~10²–10³ TCID50/mL (BinaxNOW 140 TCID50/mL nasal) | 10–15 min | 2–4 lines practical | $2–5 retail; $0.30–0.80 COGS | Yes (shipping) | Semi-quant w/ reader |
| RT-LAMP (colorimetric/fluorescent) | ~50–500 RNA copies/µL | 20–40 min | 3–4 channels (Aptitude Metrix: COVID+FluA+FluB in 20 min) | $5–15 COGS, $25–50 retail | Yes (Lucira, Aptitude Metrix EUA Feb 2025) | Real-time LAMP gives Ct-equivalent |
| CRISPR (SHERLOCK Cas13 / DETECTR Cas12) | ~10 copies/µL | 30–60 min | 2–4 | $5–20 COGS projected | Not yet at-home (Sherlock acquired by OraSure 2024, OTC trial ongoing) | Possible |
| Microfluidic real-time PCR | ~100 copies/mL | 20–30 min | 4–6 | $20–65 retail (Cue ~$65/test) | Yes (Cue, Visby, Talis) | Yes — true Ct |
| Electrochemical / impedance | ~40 TCID50/mL research | 5–30 min | Limited | <$2 sensor research | Research only | Yes |
| SERS lateral flow | fg/mL (~3 logs better than colorimetric) | 15–30 min | 4–8 plausible | Reader $$$$ | Research / lab POC | Yes |

For sub-$50 in-home use today, **only antigen LFA and RT-LAMP are real**. CRISPR is
one product cycle away. Microfluidic PCR works but cartridge/reader economics make
sub-$30 retail very hard (Cue's $249 reader + $65 cartridges sank the consumer
business model). Electrochemical and SERS are not yet shipping consumer products.

## 3. Detection chemistries — antibody / mucosal IgA & IgG

No FDA-cleared *nasal-swab antibody* product exists. Mucosal IgA assays in market
are:
- **Salimetrics** Salivary Secretory IgA ELISA and SARS-CoV-2 IgG saliva panel —
  research-use lab ELISA, plate format.
- **RayBiotech** IgA detection kits — same: lab ELISA / multiplex array.
- COVID-era nasal-IgA research assays (e.g. Mologic) — never crossed into a
  commercial kit.

Key data point: nasal-secretion spike-specific IgA is **>3× higher than saliva IgA**
and correlates well with serum (Frontiers in Immunology 2024). The analyte is real
and the matrix is favorable — **this is a defensible wedge for a startup**: adapt a
salivary SIgA ELISA chemistry onto an LFA strip, calibrate against serum gold
standard. LFA-based IgA/IgG quant has been demonstrated at picomolar–nanomolar
dynamic range for AMH, hCG, and PCT; nasal IgA at ng–µg/mL is well within reach.

## 4. Reader hardware

**Smartphone camera + LFA strip.** Cheapest path. COGS of strip+cassette $0.50–1.50;
"reader" is just app + cardboard hood/QR fiducial. Examples: Healthy.io (urinalysis,
FDA-cleared), Scanwell Health (UTI w/ Lemonaid; acquired by BD 2021), Ellume (had
integrated Bluetooth reader). Engineering: 6–12 months for app + clinical
validation.

**Dedicated reader.** Cue ($249 retail, ~$120 COGS), Lucira (single-use, ~$15 COGS,
disposed after one test), Visby (single-use disposable PCR, ~$25–40 COGS). Building
from scratch: $3–8M NRE and 18–30 months.

**Contract manufacturers.** Phillips-Medisize (CDMO, ISO 7/8 cleanrooms), Jabil
Healthcare, Flex, Plexus, Foxconn Health for high-volume. Microfluidic specialists:
Enplas, uFluidix, Vantiva, Parallel Fluidics. Lumos Diagnostics offers white-label
LFA reader development.

## 5. Cartridge / consumable design

Microfluidic IVD cartridge market: $12B in 2025, growing to $40B+ by 2030. Typical
economics:
- **NRE / tooling**: $250k–$1.5M for an injection-molded cartridge family.
- **Unit cost at volume**: $8–25 at 10k/yr, $3–8 at 100k/yr, $0.80–3 at 1M/yr.
- Equipment: precision injection molding, ultrasonic welding/laser bonding,
  conjugate-pad lamination, reagent dispensing + lyophilization.

For LFA-only, contract LFA manufacturers (Abingdon Health, Lumos, BBI Solutions, DCN
Diagnostics) can do design + manufacturing for $300k–$1M NRE and $0.30–1.20 unit
cost.

## 6. Quantification & dynamic range

Smartphone camera + LFA can produce a quantitative signal — but limits matter.
Single-line dynamic range is ~1–2 orders of magnitude. Tricks to extend it:
- **Time-series acquisition** (kinetic capture at t = 5, 10, 15 min): slope is more
  linear than endpoint, adds ~0.5–1 log.
- **Multiple test lines at different antibody concentrations** ("ladder" strips):
  each saturates at a different titer; AI bins by which line crosses threshold.
- **Internal calibrator line + reference color patch** on cassette: corrects for
  ambient light, camera, lot variation.
- **Dual-modality** (fluorescent + colorimetric on one strip): another log of range;
  needs UV illumination.

For early/peak/recovering binning: ~3–4 distinguishable bins of antibody titer plus
a co-reported pathogen-load signal. **A 2-strip cassette (one antigen + one IgA
ladder) read kinetically by phone is the minimum viable architecture** for
stage-of-infection inference.

## 7. End-to-end architecture options

**Option A — Paper strip + phone (cheapest).** AN foam swab → buffer tube → dual LFA
cassette (antigen + 3-line IgA ladder) → phone-camera kinetic read with cardboard
hood. **COGS $1.50–3, retail $15–25.** Detects 1–2 pathogens + relative IgA titer +
crude stage. Best AI-signal/$ ratio. 12–18 months to MVP, $1–2M NRE.

**Option B — Disposable reader + cartridge (mid).** Single-use injection-molded
cartridge with LAMP or CRISPR + integrated heater + photodiode (Lucira-style).
**COGS $8–15, retail $40–60.** 3–4 pathogens at PCR-grade sensitivity + crude
antibody if IgA channel added. 24–36 months, $5–10M NRE.

**Option C — Reusable reader + cartridge + cloud (ambitious).** Bluetooth reader
($80–150 COGS, $200–300 retail) + multiplex microfluidic cartridge ($10–25 COGS).
Cue/Visby tier. Multi-pathogen + quantitative IgA/IgG. 36–48 months, $15–30M NRE
plus clinical trials.

## 8. Build vs partner

Three precedents:
- **Software-on-someone-else's-strip** (Scanwell licensed BD's strip; Healthy.io
  OEMs Siemens dipsticks). Fastest, but you depend on partner FDA filings and lose
  differentiation. Scanwell got acquired *because* the strip-owner has all the
  leverage.
- **OEM the cartridge, build reader/app** (Lucira, Aptitude Metrix). More capital,
  more control.
- **Full vertical** (Cue, Visby) — $100M+ raise required.

For a <$2M startup, **partner for the strip, own the AI + workflow**. Healgen (FDA
De Novo for at-home COVID/Flu Sept 2024), QuidelOrtho (QuickVue family), and SD
Biosensor are open to OEM/private-label. Add a custom IgA-detection strip via a
contract LFA shop and build smartphone-vision quantification in-house — that's the
differentiated piece.

## 9. Bottom line — recommendation for <$2M MVP

**Bet: dual-strip LFA cassette + smartphone camera + AI quantification.**
- **Strip 1**: licensed/OEM antigen LFA (COVID + Flu A/B + RSV multiplex from
  Healgen or SD Biosensor).
- **Strip 2**: custom mucosal IgA ladder (3 test lines at calibrated capture-Ab
  concentrations) developed with a contract LFA manufacturer; ~$300–600k NRE, 9–12
  months.
- **Reader**: phone camera + printed cassette with fiducials and color calibration
  patch + cardboard hood. App captures kinetic time series; AI infers pathogen
  presence, IgA titer bin, and stage of infection.
- **COGS target**: <$3/test. **Retail**: $20–30. **MVP**: 12–18 months. **Capital**:
  $1.5–2M (~$600k strip dev, $400k app/AI, $300k clinical validation, $500k
  regulatory + working capital).

Why this wins per dollar of complexity: the AI gets the richest input signal
(kinetic, multi-line, dual-analyte) without the company taking on cartridge/reader
hardware risk. The IgA-quantification claim — the actual differentiator — stays
in-house and patentable.

Avoid for MVP: custom reader (Cue's failure mode), CRISPR (not OTC-shippable yet),
electrochemical/SERS (research-only), microfluidic PCR (capital-intensive).

## Sources

- [PLOS One — AN vs NP swab meta-analysis](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0254559)
- [PMC — NP / MT / OP clinical evaluation](https://pmc.ncbi.nlm.nih.gov/articles/PMC8675123/)
- [FDA — Lucira CHECK-IT EUA fact sheet](https://www.fda.gov/media/147493/download)
- [FDA — BinaxNOW Self Test EUA fact sheet](https://www.fda.gov/media/147254/download)
- [Copan FLOQSwabs product page](https://www.copanusa.com/products/flocked-swabs-traditional-swabs/floqswabs-flocked-swabs/)
- [PMC — foam vs flocked anterior-nares comparison](https://pmc.ncbi.nlm.nih.gov/articles/PMC2832461/)
- [PubMed — lyophilized colorimetric RT-LAMP at-home kit](https://pubmed.ncbi.nlm.nih.gov/35487969/)
- [Aptitude Metrix COVID/Flu EUA (Feb 2025)](https://www.prnewswire.com/news-releases/aptitude-receives-fda-authorization-for-metrix-covidflu-multiplex-molecular-test-for-point-of-care-and-over-the-counter-use-302382621.html)
- [Mammoth Biosciences diagnostics](https://mammoth.bio/diagnostics/)
- [GenEngNews — Sherlock Cas12 / OraSure acquisition](https://www.genengnews.com/gen-edge/elementary-claim-sherlock-gains-patent-rights-for-cas12-in-diagnostics/)
- [PMC — FDA-authorized molecular POC SARS-CoV-2 review](https://pmc.ncbi.nlm.nih.gov/articles/PMC9122839/)
- [Fortune — Cue Health business-model failure](https://fortune.com/2022/05/19/cue-health-pcr-covid-19-test-device-failure-consumers/)
- [Visby — at-home STI PCR](https://visby.com/)
- [Salimetrics — Salivary SIgA ELISA](https://salimetrics.com/assay-kit/salivary-secretory-iga-elisa-kit/)
- [Frontiers in Immunology 2024 — nasal vs saliva vs serum spike IgA/IgG](https://www.frontiersin.org/journals/immunology/articles/10.3389/fimmu.2024.1346749/full)
- [PMC — Smartphone LFA reader technologies review](https://pmc.ncbi.nlm.nih.gov/articles/PMC9571991/)
- [Phillips-Medisize IVD capabilities](https://phillipsmedisize.com/in-vitro-diagnostics/)
- [TN Plastics — microfluidics cartridge market 2025–2030](https://www.tn-plastics.com/post/microfluidics-cartridge-injection-molding-the-next-frontier-in-diagnostics-life-sciences-2025)
- [Fierce Biotech — BD acquires Scanwell](https://www.fiercebiotech.com/medtech/bd-acquires-smartphone-powered-covid-test-partner-scanwell-health)
- [Healgen FDA De Novo at-home COVID/Flu (Sept 2024)](https://www.prnewswire.com/news-releases/healgen-scientific-receives-fda-de-novo-marketing-authorization-for-at-home-covid-19-and-influenza-test-302269927.html)
- [PMC — Miniaturized Raman / SERS POC for respiratory viruses](https://pmc.ncbi.nlm.nih.gov/articles/PMC9405795/)
