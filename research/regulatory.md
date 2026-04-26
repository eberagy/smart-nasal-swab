# Regulatory, Data & Privacy Memo

**Scope:** Regulatory + data + privacy only. The product = (i) a self-collected
nasal-swab IVD that does multiplex pathogen ID + quantitative antibody titer, plus
(ii) an AI/LLM layer that interprets the strip and outputs a treatment
recommendation. Both layers need separate FDA authorization. This memo flags the
choices that, if mis-scoped, will kill the company.

---

## 1. FDA pathway for the assay

The assay is two devices stitched together: a **multiplex molecular respiratory
panel** (qualitative pathogen ID, regulated under 21 CFR 866.3980, product code
**OOI/PZF**) and a **quantitative serology assay** (regulated under 21 CFR 866.3xxx
series). Each has different regulatory weight.

- **Multiplex pathogen ID portion (510(k) likely available).** The De Novo creating
  this category was BioFire's FilmArray Respiratory Panel (DEN170017) and RP2.1
  (DEN200031). After De Novo, BioFire RP2.1 became a **predicate**, and follow-ons
  (BIOFIRE SPOTFIRE, Cepheid Xpert Xpress, Visby) cleared via 510(k). For an at-home
  OTC version specifically, **Cue Health's COVID-19 Molecular Test was the first De
  Novo for an OTC molecular test (DEN200094)**; **Lucira's COVID + Flu OTC was the
  first multiplex at-home (EUA → De Novo).** A new at-home multiplex respiratory +
  serology product likely needs a **fresh De Novo** for the OTC + multi-analyte
  intended use, because no on-market predicate covers OTC quantitative antibody
  titer.
- **Quantitative antibody titer portion (this is the hard part).** Quantitative
  serology that purports to *stage* infection (acute vs. convalescent vs. immune) is
  not a settled category. Most cleared serology assays are qualitative or
  semi-quantitative; the FDA has historically been reluctant to permit OTC
  quantitative serology, and SARS-CoV-2 serology revocations in 2024–2025 reinforced
  this. Expect either **De Novo with stringent special controls** or, if claims
  include "treatment-guiding," a slide toward **Class III / PMA**.
- **Combined POC + quantitative + treatment claim.** Realistically: **De Novo** for
  the IVD as a whole, with mandatory clinical performance studies in the intended
  use environment (home, lay user). Class III / PMA is reserved for life-supporting
  or high-risk devices; FDA has signalled (HBV reclassification, 90 FR 40285, Sept
  2025) that multi-analyte infectious-disease assays are moving *down* to Class II,
  not up — but treatment-directing claims pull risk back up.
- **Cost / time (FY2025 MDUFA V).** De Novo standard fee: **$162,108** (small
  business **$40,527**); 510(k) standard fee **$24,335** (small business **$6,084**);
  PMA standard **$540,361**. Realistic clock: De Novo ~10–18 months from filing (FDA
  target 150 days but the average for novel IVDs is 9–12+ months); PMA 2–4 years;
  clinical study build + run adds 12–24 months upstream. Total program cost for the
  IVD half alone: **~$8–25M** depending on whether you outsource the wet lab.

## 2. FDA pathway for the AI/ML layer (SaMD)

The AI is a separate device function under FDA's Sept 27, 2022 CDS final guidance
and the **Dec 4, 2024 final PCCP guidance** ("Marketing Submission Recommendations
for a Predetermined Change Control Plan for Artificial Intelligence-Enabled Device
Software Functions") together with the **Jan 6, 2025 draft guidance** "Artificial
Intelligence-Enabled Device Software Functions: Lifecycle Management and Marketing
Submission Recommendations."

- **Strip-image interpretation → pathogen + titer readout.** SaMD that automates an
  IVD readout is a device. By IMDRF risk framework + FDA practice it is **Class II**
  (moderate risk, drives clinical management of a non-serious condition) → cleared
  via **510(k) with PCCP**. Predicates exist (Healthy.io Minuteful Kidney; Scanwell;
  numerous lateral-flow image readers).
- **Antibody-stage estimator.** If it merely outputs a number with reference ranges,
  **Class II**. The moment it outputs "you are in acute phase, you have ~5 days of
  viral shedding remaining" — that's diagnostic interpretation that drives treatment
  timing → still Class II, but the special controls become much heavier
  (representative training data, subgroup performance, prospective validation).
- **Treatment recommendation.** This is the inflection point. A SaMD that recommends
  a specific therapy (e.g., "take oseltamivir 75 mg BID x 5 days") is no longer a
  CDS carve-out (see §3) and is no longer plain Class II. FDA will treat this as
  **Class II with the very real possibility of Class III / De Novo** depending on
  (a) whether it recommends a controlled or Rx-only drug, (b) whether the clinician
  is the user vs. the patient, (c) whether it can be overridden. Expect **De Novo +
  PCCP**, mandatory locked-version vs. adaptive-component delineation, and
  human-in-the-loop requirements.
- **PCCP mechanics.** Per the Dec 2024 final guidance, the PCCP must contain (i)
  Description of Modifications, (ii) Modification Protocol, (iii) Impact Assessment.
  This is the single most important regulatory artifact for an LLM-based product,
  because without it every model retraining is a new 510(k). Build the PCCP into the
  v1 submission.

## 3. CDS carve-out under §3060(a) of the 21st Century Cures Act

The four statutory criteria (FD&C Act §520(o)(1)(E), as amended) for being
**non-device CDS**:
1. Not intended to acquire/process/analyze a medical image or signal from an IVD;
2. Intended to display, analyze, or print medical info;
3. Intended to support clinician (not patient) recommendations;
4. **Enables clinician to independently review the basis** for the recommendation.

A nasal-swab AI fails criterion #1 immediately — it processes IVD output. It also
fails #3 if the recommendation goes to the patient OTC, and #4 if the model is an
LLM whose reasoning isn't transparently surfaced. **The CDS carve-out is not
available.** This is a device. Plan around that, don't fight it.

## 4. CLIA / CLIA waiver

Three layers, governed by 42 CFR Part 493:
- **Lab-based / professional use (CLIA-moderate or high complexity)** — fastest
  path, but irrelevant if you want at-home.
- **CLIA-waived (POC)** — pharmacy, urgent care, school nurse. Requires a **CLIA
  Waiver by Application** (or **Dual Submission** combining 510(k)/De Novo + waiver).
  FDA's Jan 2020 guidance "Recommendations for Clinical Laboratory Improvement
  Amendments of 1988 (CLIA) Waiver Applications" governs. Standard requires
  demonstrating "insignificant risk of erroneous result" with untrained users.
  Roche's Apr 2025 cobas liat pertussis is the recent template.
- **At-home OTC** — needs explicit OTC labeling indication. Cue's De Novo
  (DEN200094) and Lucira's De Novo are the playbook: lay-user usability validation
  per IEC 62366-1, label comprehension studies, performance studies in untrained
  users, telehealth/result-reporting plan. CLIA does not technically apply to true
  at-home tests sold direct to consumers (the consumer isn't a "laboratory"), but
  **state public-health reporting still does** (see §5), and the FDA's OTC
  authorization has effectively absorbed the CLIA-waiver-equivalent rigor.

## 5. HIPAA, state law, GDPR/IVDR

- **HIPAA.** A direct-to-consumer test company that does *not* bill insurance and
  has no provider relationship is generally **not a HIPAA covered entity**. But the
  moment you (a) connect with a telehealth physician for prescribing, (b) bill
  payors, or (c) act as a Business Associate to a covered lab, HIPAA attaches.
  Realistic plan: assume HIPAA applies, structure a **hybrid entity** with the
  consumer-data side governed by the FTC Health Breach Notification Rule (16 CFR
  Part 318, expanded April 2024 to cover health apps explicitly) and the clinical
  side under HIPAA. Note: 23andMe and Flo Health enforcement actions show FTC will
  pursue you under §5 + HBNR even without HIPAA.
- **CLIA-mandated patient access.** 42 CFR 493.1291(l) (per CMS-2319-F, Feb 2014)
  requires the lab to give the patient direct access to test reports — even without
  going through a physician.
- **State public-health reporting.** Influenza, COVID, RSV, syphilis, gonorrhea,
  chlamydia, pertussis, measles are reportable in every state under state
  public-health law (lists vary; CSTE national notifiable list is the floor). HIPAA
  expressly permits these disclosures (45 CFR 164.512(b)). At-home test makers route
  reporting via a CLIA-certified partner lab or directly via state ELR (Electronic
  Lab Reporting) interfaces. Build this on day one — it is non-optional and easy to
  forget.
- **EU.** Under **IVDR 2017/746**, a multi-analyte respiratory + serology assay sold
  to lay users falls in **Class C** (Rule 3, communicable agents; possibly Class D
  if syphilis or HIV ever included). Class C requires a Notified Body + EU Reference
  Laboratory verification. **EU AI Act** (in force Aug 1, 2024; full medical-device
  provisions Aug 2, 2027) classifies device-embedded AI as **high-risk** under
  Article 6 + Annex I. **GDPR** Article 9 special-category data; explicit consent +
  DPIA required.

## 6. Data sourcing & IRB

You need (a) tens of thousands of paired strip-image + reference-PCR/serology
samples, (b) longitudinal nasal antibody trajectories, (c) outcome-linked treatment
data. Legal options:

- **Run your own clinical study (best for FDA).** IRB-approved, prospective,
  multi-site (≥3 sites, geographic + demographic diversity per FDA's Diverse
  Populations guidance, June 2024). Required for the pivotal — the FDA will not
  accept retrospective real-world data alone for a novel IVD De Novo. Budget $5–15M
  and 12–18 months.
- **Academic medical center partnerships under DUAs / sponsored research /
  IRB-cleared protocols.** Best for the AI training set; cheaper than a sponsored
  trial. Use Common Rule (45 CFR 46) compliant consent, ideally a master
  broad-consent biobank model. UPenn, Hopkins, Mayo, Stanford all have
  respiratory-virus biobanks.
- **Purchased de-identified data (Truveta, Komodo, TriNetX, Optum, HealthVerity).**
  HIPAA Expert Determination de-identified per 45 CFR 164.514(b) — usable for AI
  training without IRB if the deal restricts re-identification. Useful for
  outcome-linked treatment data. **Cannot** substitute for the FDA pivotal because
  the sample type isn't yours and reference labels aren't paired. State laws
  (Washington My Health My Data Act, California CMIA + AB 2013, Texas HB 4) further
  restrict use even of de-identified data — review each contract.
- **Synthetic data.** OK for unit testing and class balancing. **Not OK** for FDA
  pivotal evidence; FDA's Jan 2025 draft guidance explicitly warns against synthetic
  data as primary clinical evidence for AI devices.

## 7. AI bias / equity / transparency

- **FDA**: Oct 2021 Good Machine Learning Practice (10 guiding principles), June
  2024 Transparency for ML-Enabled Medical Devices guiding principles, and the Jan
  2025 draft Lifecycle Management guidance — all require representative training
  data, reporting of subgroup performance (sex, age, race/ethnicity, comorbidities),
  and a public-facing summary.
- **ONC HTI-1 final rule (89 FR 1192; effective Feb 8, 2024; compliance Dec 31,
  2024)** — adds 45 CFR 170.315(b)(11) Decision Support Interventions criteria. If
  your AI is surfaced through certified EHRs you must publish **31 source-attribute
  disclosures** ("nutrition label" / "model card") covering intended use, training
  data, validation, fairness/equity metrics, ongoing monitoring (the **FAVES**
  principles: Fair, Appropriate, Valid, Effective, Safe). Not legally required for
  direct-to-consumer apps, but de facto standard — investors and partners will ask.
- **California AB 2013** (effective Jan 1, 2026) requires public disclosure of
  training-data summaries for any generative-AI-trained product available to
  Californians. **Colorado AI Act** (SB 24-205, effective Feb 1, 2026) imposes
  risk-management duties on high-risk AI deployers, which a treatment-recommending
  product is.

## 8. Reimbursement

Likely CPT codes: **87636** (SARS-CoV-2 + Flu A/B multiplex), **87637** (+ RSV),
**87633** (≥5 respiratory targets), **86xxx** serology depending on analyte (e.g.,
86762 rubella, 86790 viral antibody NOS), plus a **PLA code** is realistically
required for a unique multiplex (apply via AMA quarterly). Medicare CLFS rates 2025:
87636 ≈ $142.63; 87637 ≈ $416.78; 87633 ≈ $416.78. Medicare does not separately
reimburse SaMD interpretation today — there is no CPT code for "AI strip read." MAC
Local Coverage Determinations (Novitas A58575) require medical necessity and limit
panel use; payers reject "panel testing without documented clinical justification"
routinely. **Bottom line: payor-funded is feasible only through a CLIA-certified
partner lab with an LCD-favorable MAC; a direct-to-consumer model should price the
test as cash-pay $40–80 and treat reimbursement as upside.**

## 9. Liability

- **Manufacturer** (you) — strict products-liability exposure for the IVD. *Dickson
  v. Dexcom* (E.D. La. 2024) tested whether De Novo authorization preempts
  state-law tort claims under *Riegel v. Medtronic* (552 U.S. 312, 2008); courts
  have largely held De Novo does **not** confer Riegel-style preemption. PMA *does*.
  This is a real argument for spending the extra money on PMA where defensible.
- **Prescriber / telehealth physician** — covered by the **learned intermediary
  doctrine** in most states; failure-to-warn claims route through them.
  AI-as-intermediary is the open question — *Tschider, "Healthcare AI's Unlearned
  Intermediaries" (2024)* and *Brandon J. Broderick* tracking show a 14% YoY rise in
  AI-related malpractice claims through 2024, with manufacturers increasingly named
  as direct defendants when the AI made the recommendation.
- **Software developer** — at greatest risk if the LLM is the proximate
  decision-maker. Mitigations: (i) require clinician sign-off before any Rx
  recommendation reaches the patient; (ii) keep the LLM "advisory" and surface
  confidence + reasoning; (iii) maintain product-liability insurance ($25M+
  aggregate); (iv) state-law disclosure under California AB 2013 / Colorado AI Act.
- **Patient direct-action under HBNR** (FTC) — civil penalties up to $51,744 per
  violation per day for breaches affecting 500+ individuals.

## 10. Bottom line — recommended strategy

**24-month roadmap**
1. **Q1–Q2 2026:** Incorporate; raise seed; engage regulatory counsel (Hogan Lovells
   / Goodwin / Ropes & Gray IVD practice). File a **Q-Sub (pre-submission)** with
   FDA CDRH OHT7 (microbiology) requesting a **classification meeting** plus a
   **Breakthrough Device designation** request for the multiplex + quantitative
   serology + AI combo (the program's 60-day decision clock makes this cheap to
   attempt).
2. **Q3 2026:** Launch a **Research Use Only (RUO)** version under 21 CFR 809.10(c)
   — strict labeling, no clinical claims, no patient-facing results. Use it to (a)
   build the strip-image dataset under IRB at 3–4 academic medical centers, (b)
   generate the analytical performance package, (c) train the v1 model with a
   **locked PCCP envelope** drafted in parallel.
3. **Q4 2026 – Q2 2027:** Pivotal clinical study (lay-user, multi-site, ≥1,500
   paired specimens with reference PCR + reference serology). Parallel CLIA-waiver
   or OTC usability per IEC 62366-1.
4. **Q3 2027:** **De Novo + PCCP submission** for the IVD-with-AI as a single
   combination device function. Use Breakthrough Sprint discussions to reduce
   review cycles.
5. **Q1 2028:** Authorization; soft launch in 2–3 states; activate ELR reporting;
   layer in payor-billed CLIA-lab variant via a partner.

**5-year roadmap**
- Years 3–4: First 510(k) line-extension under the PCCP (more pathogens; updated
  model). Begin **EU IVDR Class C** Notified Body engagement (BSI, TÜV SÜD); align
  with EU AI Act high-risk obligations before Aug 2027.
- Year 5: Second-generation device — if quantitative serology + treatment
  recommendation has held up safely, pursue a Class III PMA upgrade where treatment
  recommendation is the labeled use, to gain Riegel preemption against state tort
  claims and command premium reimbursement.

**The single biggest do-not-do:** Do NOT ship a product that auto-recommends
antivirals or antibiotics OTC without prescriber-in-the-loop in v1. That single
design choice converts a hard-but-tractable Class II De Novo into a likely Class III
PMA *and* exposes you to learned-intermediary-doctrine collapse. Keep the LLM
advisory until you have post-market data.

**The single biggest must-do:** Build the **PCCP** and the **subgroup-performance /
FAVES disclosure** into the v1 architecture. Both are now the gating artifacts FDA
reviewers and ONC-certified channel partners are scoring you on.

---

## Sources

- [FDA — Marketing Submission Recommendations for a PCCP for AI-Enabled Device Software Functions (final, Dec 2024)](https://www.fda.gov/regulatory-information/search-fda-guidance-documents/marketing-submission-recommendations-predetermined-change-control-plan-artificial-intelligence)
- [FDA — Final guidance PDF (PCCP)](https://www.fda.gov/media/166704/download)
- [FDA — Draft guidance, AI-Enabled Device Software Functions: Lifecycle Management and Marketing Submission Recommendations (Jan 6, 2025)](https://www.fda.gov/media/184856/download)
- [FDA — Changes to Existing Medical Software Policies Resulting from Section 3060 of the 21st Century Cures Act (final, Sept 28, 2022)](https://www.fda.gov/regulatory-information/search-fda-guidance-documents/changes-existing-medical-software-policies-resulting-section-3060-21st-century-cures-act)
- [Federal Register — CDS Software final guidance availability notice (87 FR 58775, Sept 28, 2022)](https://www.federalregister.gov/documents/2022/09/28/2022-20993/clinical-decision-support-software-guidance-for-industry-and-food-and-drug-administration-staff)
- [FDA — De Novo classification, BioFire Respiratory Panel 2.1 (DEN200031)](https://www.accessdata.fda.gov/cdrh_docs/pdf20/DEN200031.pdf)
- [FDA — At-Home OTC COVID-19 Diagnostic Tests landing page](https://www.fda.gov/medical-devices/coronavirus-covid-19-and-medical-devices/home-otc-covid-19-diagnostic-tests)
- [FDA — Recommendations for CLIA Waiver Applications (final, Jan 2020)](https://www.fda.gov/regulatory-information/search-fda-guidance-documents/recommendations-clinical-laboratory-improvement-amendments-1988-clia-waiver-applications)
- [FDA — Breakthrough Devices Program guidance (final, Sept 2023)](https://www.fda.gov/files/guidance%20documents/published/Breakthrough-Devices-Program.pdf)
- [HHS / CMS — CLIA Program and HIPAA Privacy Rule; Patients' Access to Test Reports (CMS-2319-F, 79 FR 7290, Feb 6, 2014)](https://www.federalregister.gov/documents/2014/02/06/2014-02280/clia-program-and-hipaa-privacy-rule-patients-access-to-test-reports)
- [HHS — Summary of the HIPAA Privacy Rule](https://www.hhs.gov/hipaa/for-professionals/privacy/laws-regulations/index.html)
- [ONC — HTI-1 Final Rule overview](https://www.healthit.gov/topic/laws-regulation-and-policy/health-data-technology-and-interoperability-certification-program)
- [Federal Register — HBV Reclassification (90 FR 40285, Sept 18, 2025)](https://www.federalregister.gov/documents/2025/09/18/2025-18082/microbiology-devices-reclassification-of-antigen-antibody-and-nucleic-acid-based-hepatitis-b-virus)
- [Ropes & Gray — Final Guidance on CDS Software (Oct 2022)](https://www.ropesgray.com/en/insights/alerts/2022/10/is-your-clinical-decision-support-software-a-medical-device)
- [Ropes & Gray — FDA Finalizes Guidance on PCCPs (Dec 2024)](https://www.ropesgray.com/en/insights/alerts/2024/12/fda-finalizes-guidance-on-predetermined-change-control-plans-for-ai-enabled-device)
- [Winston & Strawn — AI and the Learned Intermediary Doctrine](https://www.winston.com/en/blogs-and-podcasts/product-liability-and-mass-torts-digest/a-new-intermediary-artificial-intelligence-and-the-learned-intermediary-doctrine)
- [Drug & Device Law Blog — Possible Learned Intermediary Showdown (Aug 2024, Dickson v. Dexcom)](https://www.druganddevicelawblog.com/2024/08/possible-learned-intermediary-showdown-in-michigan.html)
- [Reed Smith — The EU AI Act and Medical Devices](https://www.reedsmith.com/our-insights/blogs/viewpoints/102kq35/the-eu-ai-act-and-medical-devices-navigating-high-risk-compliance/)
- [CMS — Article A58575, Billing & Coding: Respiratory Pathogen Panel Testing](https://www.cms.gov/medicare-coverage-database/view/article.aspx?articleId=58575)
