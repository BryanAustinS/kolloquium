# Speaker Notes — Kolloquium (20 min target)

Timing plan: ~1 min per slide, results slides get the extra time.
Budget: intro block (1–4) ≈ 4 min · method block (5–9) ≈ 6 min · results block (10–13) ≈ 6 min · answers & closing (14–17) ≈ 4 min.

---

## 1 · Title (0:30)

- Good afternoon. My thesis: **hybrid plausibility checking of aviation flight data** — comparing a rule-based check with a machine-learning classifier on the same real data.
- Carried out at datatactics in cooperation with Lufthansa Systems.

## 2 · Why Flight-Data Plausibility? (1:00)

- Aviation is one of the most data-intensive industries: new-generation aircraft produce terabytes per flight.
- This data feeds the Lufthansa group's fuel data warehouse — datatactics runs the ETL. Downstream: **ETS emissions audits** open to external authorities, and fuel-efficiency analysis.
- Point to land: an audit is only as good as the records that go into it. Plausibility asks: *is this value credible in its context?*

## 3 · Current Practice (1:00)

- Today: a hand-written Groovy rule engine, checking **only fuel columns** — e.g. the fuel mass balance within 200 kg.
- Three classic weaknesses: brittle, expensive to change (release per change), incomplete — the unanticipated slips through.
- The 10,000 kg example: schema-valid, contextually absurd. That's the gap plausibility checking fills.
- If asked about related work: no prior system combines rule-based + ML plausibility checking on tabular flight data (Jasra et al. 2025 = ML on flight data, no rules; Mohite & Ouarbya 2024 = hybrid, not aviation).

## 4 · Research Questions (0:45)

- RQ1 is *descriptive*: how far do the two approaches differ on the same data?
- RQ2 is the *practical* question the company cares about: can ML replace the hand-written rules, or only complement them?
- Plus the engineering goal — the configurable web app. (Mention once; the talk focuses on the ML results.)

## 5 · Approach (1:00)

- Two stages. Stage 1 (Module A): statistical detector + rule engine → labels every record clean / explained / **unexplained anomaly**.
- Stage 2 (Module B): a supervised classifier **trained on those labels**.
- Key framing sentence: *the rule stage generates the training signal for the learned stage* — they're stages of one pipeline, not rivals.
- Methodology: Design Science Research — build, then evaluate.

## 6 · Stage 1 — How a Record Gets Its Label (1:30)

- Clusters make records comparable — a take-off weight is only unusual *relative to the same route, aircraft type, year*.
- IQR fence: robust, outliers can't drag the bounds. Chosen over 3σ together with the domain experts — 3σ proved too wide in practice.
- Walk the three flights: inside bounds → clean; ferry flight flagged but the ferry rule explains it; third flight flagged, **no rule fires → unexplained anomaly**.
- These unexplained anomalies are what the whole system is hunting.

## 7 · Data & Ground Truth (1:00)

- One confidential export: ~3 GB, ~1.4M legs, 2025–2026. After cleaning: **654,053 labelled records**.
- Separate 500 MB file with almost entirely **new routes** held back as the honest generalisation test.
- Class distribution is the core difficulty: 95% clean, target class only **2.69%**.

## 8 · Learning Setup (1:00)

- Multiclass: 5 classes, target = unexplained anomaly. Keeping the rule classes teaches the model to separate "anomalous" from "explained".
- Three models: Dummy = no-skill floor; Logistic Regression = interpretable linear baseline (supervisor's suggestion); LightGBM = standard for tabular data.
- Features: from ~200 raw columns to 100 used — half of them **engineered domain features** (residuals, physical identities, detour ratio). MLflow tracks every run.

## 9 · Training for Rare Anomalies (1:00)

- Imbalance handling: undersampling clean to 10:1 — inside CV folds only. Class weights and SMOTE both tried and rejected (precision collapse / synthetic noise).
- Tuning: grid search for LogReg; Optuna for LightGBM.
- **Threshold 0.9, precision-first** — the cost argument: a miss is cheap (retrospective checks), a false alarm is expensive (manual review, trust). This is a deliberate business decision, be ready to defend it.
- Metric: PR-AUC, because accuracy is meaningless at 2.7% positives. No-skill = 0.027.

## 10 · Results — Building the Model Step by Step (2:00) — INTERACTIVE

**This slide advances by clicking on the chart** (5 clicks). Each click grows the next bar; the caption box explains the lever. After the 5th step, one more click moves to the next slide. The "restart" link (bottom left) resets it.

1. **Dummy 0.027** — "any model must clearly beat this floor."
2. **LogReg 0.131** — "even tuned — a straight line can't combine features; tuning moved it by 0.002."
3. **LightGBM raw 0.317** — "default settings, raw columns — already 12× the floor. The signal is in the data; only trees can use its interactions."
4. **+ features 0.308** — pause on the DIP: "features alone did nothing — the default model can't exploit them."
5. **+ tuning 0.493** — "tuning unlocks the features. 3.8× Logistic Regression, 18× no-skill. This is the final model."
- Mention while final bar shows: more training data was still improving cross-file performance — data volume was the other big lever (0.33 → 0.53 precision on the unseen file going 500 MB → 2 GB).

## 11 · The Final Model (1:30)

- Per-class shape: clean nearly solved (F1 0.98); common rule classes fine (0.67 each); **ZFW OK unlearnable (F1 0.04, only 259 examples)** — the deterministic rule wins there.
- Target class: **precision 0.76, recall 0.24**. When it flags, it's right 3 times out of 4; it finds 1 in 4.
- Confusion matrix: errors are almost all "anomaly predicted clean" — it almost never confuses anomalies with rule classes.

## 12 · Choosing the Operating Point (1:00)

- The threshold sweep is stored — precision and recall trade smoothly, no retraining needed to move the operating point.
- At 0.9: ~¾ of the review queue is real. That's what keeps the tool credible to analysts.
- If recall ever matters more (e.g. audit season), drop the threshold — the curve shows exactly what you get.

## 13 · Generalisation (1:15)

- Honest range: **0.49 on known routes, 0.30 on new ones** — a third lower where routes barely overlap.
- The ablation story is worth telling: group-keyed features hit 0.80 in-distribution and **collapsed to 0.05** across files — they memorised the groups. Excluded. This is why the reported numbers are trustworthy.
- Cross-file performance had **not plateaued** with data volume — floor, not ceiling.

## 14 · RQ1 Answer (1:15)

- Agreement: 96.6% of records. On frequent, rule-shaped patterns the model reproduces the rules almost exactly.
- Divergence concentrates in three places: unexplained anomalies (soft boundary, ~¼ recovered), decision *nature* (hard explainable yes/no vs probability), scarce classes (rules win outright).
- The 865 extra model flags: 98% land on records the rules call clean — **candidate anomalies in the majority no rule inspects**. That's the potential added value.

## 15 · RQ2 Answer (1:15)

- Four numbered reasons point away from replacement: rule-mimicry ceiling, 24% recall, unlearnable scarce classes, off-distribution degradation.
- The converse also fails — rules can't do soft multivariate cases. **Hence: complement.**
- Be explicit about what's still open: whether the extra flags are *real* — that's the blind expert review (~6,100 records, Model 1 / Model 2 anonymised). The metrics structurally *cannot* answer it because the rules are the ground truth.

## 16 · Limitations & Future Work (1:00)

- Lead with the honest ones: single export, one airline group, three rules; labels cap what the model can discover; random (not temporal) split.
- Future: unsupervised detection to escape the label ceiling, more data, more rules → richer labels, temporal validation, finish the expert review.

## 17 · Conclusion (0:45)

- Three contributions: the architecture, the empirical comparison, the working artefact.
- Closing sentence: *rules and ML are not rivals — two stages of one pipeline. The rules stay the trusted, explainable core; the model surfaces candidates in the 95% no rule ever inspects.*
- Thank you.

---

## Transition phrases

- 4→5: "So how do we actually combine them?"
- 9→10: "So what did all of this buy us? Let me build the result up the same way we built the model."
- 13→14: "With those numbers in hand, back to the research questions."
- 15→16: "To be fair about what these results can and cannot claim…"
