# Defense Q&A Cheatsheet

Every number in the deck, plus the questions examiners are most likely to ask.

## Key numbers (memorise)

| Quantity | Value |
|---|---|
| Export | ~3 GB CSV, 2025–2026, ~1.4M legs, ~200 columns |
| Labelled training set | 654,053 records |
| Unseen test file | 149,176 records, 3.5% anomalies, almost only new routes |
| Class distribution | clean 94.99% · anomaly 2.69% · Low Cargo 2.06% · Ferry 0.21% · ZFW OK 0.04% |
| Features | 100 = 39 raw numeric + 50 engineered + 11 one-hot service types |
| Final LightGBM (in-dist) | PR-AUC 0.493 · P 0.76 · R 0.24 · F1 0.36 @ 0.9 |
| Final LightGBM (unseen) | PR-AUC 0.304 · P 0.52 · R 0.18 · F1 0.27 @ 0.9 |
| LogReg best | PR-AUC 0.131 (tuning changes almost nothing) |
| Dummy | PR-AUC 0.027 = anomaly rate |
| Ratios | LightGBM ≈ 3.8× LogReg ≈ 18× no-skill |
| Confusion (in-dist) | TP 1,252 · FN 4,028 · FP 397 · TN 190,539 |
| Confusion (unseen) | TP 944 · FN 4,254 · FP 865 · TN 143,113 |
| Agreement (unseen) | 96.6% · model-only flags 865 (0.6%) · rule-only 4,254 (2.9%) |
| Model-only flags on "clean" | 389 of 397 = 98% (30% test set) |
| LightGBM hyperparameters | 450 trees, 117 leaves, depth 10, lr 0.163, min_child 29 |
| Undersampling | clean → 10× largest minority (621,286 → 176,020) |
| Expert review set | ~6,100 disagreement records, blind (Model 1 / Model 2) |

## Likely questions & answers

**Why IQR and not 3σ / a learned detector?**
Recommended by the Lufthansa Systems domain experts, and empirically better: 3σ bounds were dragged so wide that genuine anomalies fell inside. Quartiles are robust — the anomalies you hunt can't move your own fence. A learned detector at this layer would remove the explainability the analysts require.

**Isn't training on rule labels circular?**
Partly — and that's acknowledged as the "rule mimicry ceiling" (Limitations). It also means the metrics can only answer the "beyond the rules" question in one direction: every extra model flag is scored as a false alarm by construction. That is why RQ2 is judged by the blind expert review, not by the labels. The model's candidate added value shows up in two ways: it generalises rules into soft multivariate boundaries (near-miss detection), and it flags records in the clean majority the rules never inspect. Escaping the ceiling entirely needs unsupervised/semi-supervised methods (future work).

**Why multiclass instead of binary anomaly-vs-rest?**
Keeping the rule classes separate teaches the model the difference between "statistically odd but explained" and "odd and unexplained". Collapsing them would make explained records look like anomalies and hurt precision on the target class.

**Why threshold 0.9? Isn't recall 24% bad?**
Deliberate, cost-based: this is a retrospective data-quality system, not a safety system. A miss just delays a catch; a false alarm costs analyst time and trust — and at 2.7% base rate, high recall would bury them in false alarms. The full sweep (0.3–0.9) is stored, so the operating point can be moved without retraining.

**Why does LightGBM beat Logistic Regression so clearly?**
Anomalies are defined by feature *combinations* (high trip fuel *for* a short distance). A linear model can only draw one straight boundary; trees compose interactions. Consistent with the tabular-data literature (Grinsztajn et al.). LogReg was insensitive to tuning (0.129→0.131) — it hit its representational ceiling.

**Why undersampling and not SMOTE / class weights?**
Measured on the unseen file: PR-AUC is nearly identical across all three (0.269/0.274/0.270) — resampling is a calibration dial, not a source of skill. Undersampling doubled recall and F1 at the fixed 0.9 threshold; SMOTE's synthetic records added nothing; class weights destroyed precision. Applied only inside CV training folds — never to held-out data.

**How do you know the model isn't leaking the detector's output?**
The IQR/statistical columns produced by Module A are explicitly excluded from the feature set. The group-keyed feature ablation is the cautionary tale: 0.80 in-distribution, 0.05 cross-file → memorisation, excluded.

**Why is the cross-file drop (0.49 → 0.30) acceptable?**
The export is ordered by route, so the unseen file is almost entirely disjoint routes — a worst case. Real deployment traffic mostly repeats known routes, so 0.49 is closer to production; 0.30 is the floor for brand-new routes. Also, cross-file performance was still rising with data volume.

**What exactly does PR-AUC = 0.49 mean here?**
Area under the precision–recall curve for the anomaly class across all thresholds. The no-skill reference is the positive rate (0.027), so 0.49 ≈ 18× better than chance. ROC-AUC would look deceptively good on 97:3 data; PR-AUC stays honest.

**Why not 5-fold CV numbers?**
CV was used during development (resampling inside folds); the reported final numbers use a 30% held-out test set plus the separate cross-file test, which is stricter and simpler to interpret.

**Why only three rules?**
Chosen from >10 airline rules to span firing frequency (frequent / moderate / rare) — deliberately testing how each approach handles common vs scarce patterns. More rules = future work; each new rule enriches Module B's training labels.

**Why is RQ2 phrased differently in the talk than in the thesis?**
The thesis words RQ2 as "replace or complement". The talk presents its operational core: *does the model detect anomalies the rules miss?* Replicating the rules has no practical value — the rule detector already exists — so the decisive part of RQ2 is the detect-beyond direction, which is exactly what the blind expert review measures. The thesis' replace-direction answer is retained as supporting evidence (recall 24%, scarce classes unlearnable, off-route degradation → the model cannot stand alone).

**Could the rules replace the model?**
No — rules are rigid, single-column, and miss the soft multivariate cases and anything unanticipated. Each covers the other's blind spot; that asymmetry is the argument for the hybrid.

**What happened to the expert review?**
Set up as a blind annotation task (spreadsheet, one flight per row, three verdicts side by side, IQR bounds as evidence, models anonymised, disagreement types balanced at the top). Results were pending at submission; report verbally if available by the defense.

**Is 200 kg on the fuel balance / ≤300 kg cargo / ≤1500 kg uplift arbitrary?**
They are expert-set operational tolerances from Lufthansa Systems practice, encoded as configurable rule parameters — editable in the app without redeployment.

**What is the practical value for datatactics today?**
(1) The configurable rule engine extends plausibility checks beyond fuel columns without releases. (2) The classifier ranks a small, high-precision review queue out of the 95% clean majority. (3) The comparison methodology and MLflow tracking give a template for future data domains.
