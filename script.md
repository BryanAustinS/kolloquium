# Talk Script — Bachelor Kolloquium (20 minutes)

Word-for-word script. The slides carry only the key numbers and claims — everything you
say is here. Directions are in *[brackets]*. At a calm pace (~130 words/min) each section
fits its time box; the running total is marked so you can check yourself against the clock.

---

## Slide 1 — Title (0:30 · total 0:30)

Good afternoon. My name is Bryan Austin Suharta, and today I am presenting my bachelor
thesis: *Hybrid Plausibility Checking of Aviation Flight Data* — a comparison of
rule-based and machine-learning approaches to anomaly detection. The work was carried
out at datatactics GmbH, in cooperation with Lufthansa Systems, and today I want to
answer one practical question: what happens when you run hand-written expert rules and
a machine-learning model on exactly the same real flight data?

## Slide 2 — Why Flight-Data Plausibility? (2:00 · total 2:30)

Aviation is one of the most data-intensive industries in the world. A new-generation
aircraft generates five to eight terabytes of data per flight, and the global fleet is
heading toward roughly ninety-eight million terabytes of operational data per year.

Within the Lufthansa group, this data comes from many source systems — operational data
from the Netline/Ops platform, fuel data from the FDDC system, and further feeds from
the group's airlines — and every source has its own format. This is where FLT comes in,
the system this thesis is embedded in: FLT is the ETL pipeline operated by datatactics.
It takes the raw XML from all these sources, parses it, unifies the formats, and stores
the harmonised records in a central fuel data warehouse, where Lufthansa users access
them through a web frontend. And what comes out of that warehouse matters: it backs the
EU emissions-trading audits that external authorities can inspect, and the group's
fuel-efficiency analyses.

Now, where do implausible values come from in such a pipeline? The literature names five
recurring causes: missing or irregular measurements, badly calibrated equipment,
different teams using different calculation methods, typos when values are entered by
hand — and, in the worst case, deliberate misreporting to make results look better.
These are the kinds of errors a plausibility check has to catch, because one single
implausible record can distort a fleet-wide statistic or end up in an audited report.

And that is what plausibility means — not "is this value the right type, is it in the
schema", but: *is this value credible in its context?* That question is the core of my
thesis.

## Slide 3 — Current Practice (1:15 · total 3:45)

How is this handled today? By a hand-written rule engine inside FLT — and it only
checks the fuel columns. A typical rule is the fuel mass balance you see here: block
fuel minus remaining-plus-uplift must stay within two hundred kilograms. Everything
else passes through completely unvalidated.

Hand-written rules have three classic weaknesses. They are brittle — every newly
discovered exception becomes another if-else branch. They are expensive — every change
requires a new software release. And they are incomplete — an anomaly that nobody
anticipated passes through silently.

To make that concrete: a trip fuel of ten thousand kilograms is a perfectly valid
number. No schema check, no type check will ever reject it. But for an A320 on a
thirty-minute leg, it is absurd. Only context reveals that — and that is exactly what
the current system cannot see.

## Slide 4 — Research Questions (0:45 · total 4:30)

This leads to my two research questions. First, the descriptive one: to what extent do
the results of a rule-based plausibility check differ from those of an ML-based
classifier, when both are applied to the same flight data? And second, the question that
actually decides whether the learned layer earns its place: does the classifier detect
anomalies that the rule-based check misses — or does it merely replicate the rules?
Replication alone would be pointless: the rule detector already exists. The value, if
there is any, lies in what the model finds beyond it — and that is precisely what the
expert review is designed to judge.

Alongside these, the thesis had an engineering goal: a configurable web application
where domain experts can run both approaches on their own data and extend the rule set
without touching production code. I will focus on the machine-learning results today
and only mention the application briefly.

## Slide 5 — Methodology (1:00 · total 5:30)

How did I work? The thesis follows Design Science Research: take a real industrial
problem, build an artefact for it, and evaluate that artefact empirically —
iteratively, with the requirements evolving together with Lufthansa Systems. The
artefact is the Streamlit application; rules and clusters live in a database and are
editable at runtime.

The evaluation has two parts. Numerically, both stages are scored on the same data:
PR-AUC against a no-skill baseline, on a held-out test set and on a file of unseen
routes. But numbers against rule labels cannot say who is right when the two
approaches disagree — so we ask the experts, and we ask them blind. One flight per
row in a spreadsheet; the rule verdict and the two model verdicts side by side,
anonymised as Model 1 and Model 2; next to each verdict a box the expert ticks when
it is correct; and as evidence, each record's values together with its group's IQR
bounds. The disagreement types are mixed evenly at the top of the file, so even a
partial review covers every kind.

## Slide 6 — Approach (1:00 · total 6:30)

My approach is a hybrid pipeline with two stages, and the key idea is how they connect.

Stage one combines a statistical detector with a rule engine. It labels every single
record as one of three things: clean, explained, or an unexplained anomaly — I will show
you exactly how on the next slide.

Stage two is a supervised classifier — LightGBM and Logistic Regression, compared
against a no-skill baseline — and it is trained *on the labels that stage one
produces*. So the rules are not competing with the model; the rules *generate the
training signal* for the model. That single sentence is the design of the whole system.

## Slide 7 — Stage 1: How a Record Gets Its Label (1:30 · total 8:00)

So how does a record get its label? Two steps.

First: context. A take-off weight is never unusual in absolute terms — only compared to
similar flights. So records are grouped into clusters — same airline, same year, same
aircraft type, same route — and inside each cluster, the detector computes the
interquartile range and flags every value outside the fence: below Q1 minus one-point-five
IQR, or above Q3 plus one-point-five IQR. We chose the IQR fence together with the domain
experts because it is robust: the anomalies we are hunting cannot widen their own fence —
unlike the three-sigma rule, which we tried first and which proved too wide in practice.

Second: explanation. These three flights are real records from one Brussels-to-Geneva
route. The first one sits inside the bounds — clean, ninety-five percent of all records.
The second is flagged: forty-eight thousand kilograms take-off weight, far too light.
But it is a ferry flight — a repositioning flight with no passengers — and an expert
rule recognises exactly that pattern, so it is *explained*, not an error. The third
flight is also flagged — and no rule fires. That is an *unexplained anomaly*, and those
records are the detection target of the entire system.

## Slide 8 — Data & Ground Truth (1:15 · total 9:15)

The data: one confidential export from Lufthansa Systems — about three gigabytes,
roughly one-point-four million flight legs. After cleaning and deduplication, stage one
labelled six hundred fifty-four thousand records for training. A separate file of about
a hundred and fifty thousand records — almost entirely different routes — was held back
as the honest generalisation test; the model never sees it during training.

And here is the shape of the problem: ninety-five percent of all records are clean. The
rule classes explain another two-point-three percent. What remains — two-point-six-nine
percent, about seventeen and a half thousand records — are the unexplained anomalies the
classifier has to find. So this is a needle-in-a-haystack problem: an extreme class
imbalance, and everything that follows is shaped by it.

## Slide 9 — Learning Setup (1:00 · total 10:15)

The learning task is multiclass classification: five classes — clean, the three rule
classes, and anomaly, the target. Keeping the rule classes separate matters: it teaches
the model the difference between "statistically odd but explained" and "odd and
unexplained".

Three models. A Dummy classifier that always predicts clean — that is the no-skill
floor any real model has to beat. Logistic Regression as the interpretable linear
baseline. And LightGBM — gradient-boosted decision trees, the standard for large
tabular data.

For features, I reduced about two hundred raw columns to one hundred used features,
half of them engineered from domain knowledge: fuel per distance, planned-versus-actual
residuals, physical identities like take-off weight minus zero-fuel-weight plus fuel —
quantities where a large value directly signals that something does not add up. Every
experiment was tracked with MLflow, so each number I show is reproducible.

## Slide 10 — Training for Rare Anomalies (1:15 · total 11:30)

Three decisions made training work on such imbalanced data.

First, undersampling: the clean majority is trimmed to ten times the largest minority
class, only inside the training folds. I also tried class weights — precision
collapsed — and SMOTE oversampling — synthetic records added nothing.

Second, tuning: grid search for Logistic Regression, and Optuna for LightGBM, which
landed at four hundred fifty trees of depth ten.

Third — and this is a deliberate business decision, not a technical one — the decision
threshold is set precision-first, at zero-point-nine. The reasoning: a missed anomaly
is cheap here. These are retrospective data-quality checks on flights that have already
landed — nothing is at risk, and a later run can still catch it. A false alarm is
expensive: every flagged record is reviewed by hand, and noise erodes the analysts'
trust in the tool.

And because ninety-seven percent of records are not anomalies, accuracy would be
meaningless — the headline metric is PR-AUC, where the no-skill floor is zero-point-
zero-two-seven.

## Slide 11 — Results: Model Comparison (2:30 · total 14:00) — INTERACTIVE

*[The chart is interactive: chips top-left toggle the models, buttons top-right switch
the metric. Start on PR-AUC, both models visible.]*

Now the results. Green is LightGBM, blue is Logistic Regression, the dashed line is the
no-skill floor, and the x-axis is the story of how the model was built: raw columns,
plus engineered features, plus tuning.

Look at the raw columns first: LightGBM starts at zero-point-three-two — already twelve
times the floor, using nothing but the raw data. Logistic Regression manages
zero-point-zero-nine. Adding the engineered features alone changes almost nothing for
either model. But tuning — that is the step that unlocks everything: LightGBM jumps to
zero-point-four-nine, while Logistic Regression stays flat at zero-point-one-three. It
is not a tuning problem — a linear model simply cannot combine features, and anomalies
are defined by feature combinations: a high trip fuel *for* a short distance. The final
gap: three-point-eight times Logistic Regression, eighteen times the no-skill floor.

*[Click "Precision".]* One more thing worth seeing. On precision, Logistic Regression's
raw model actually looks best — zero-point-six-two. *[Click "Recall".]* Until you look
at recall: that precision came at recall zero — it essentially never fired. LightGBM
ends at recall zero-point-two-four, deliberately low at our precision-first threshold.
*[Optionally click "F1", then return to "PR-AUC".]*

## Slide 12 — The Final Model (1:30 · total 15:30)

Per class, the final model looks like this. Clean is nearly solved. The two common rule
classes work well. The rare rule class — two hundred fifty-nine training examples — is
effectively unlearnable, F1 zero-point-zero-four; the deterministic rule wins there,
every time. And on the target class: precision zero-point-seven-six, recall
zero-point-two-four. In words: when the model flags a record, it is right three times
out of four — but it finds only one in four of the anomalies.

The confusion matrix shows the errors are almost all of one kind: anomalies predicted
as clean. The model almost never confuses an anomaly with a rule class — it has genuinely
learned the difference between "unusual" and "explained".

## Slide 13 — Choosing the Operating Point (0:45 · total 16:15)

This trade-off is not fixed. The full threshold sweep is stored, so the operating point
can be moved at any time without retraining. At zero-point-nine, about three quarters of everything in the review
queue is a real anomaly — a small, credible queue that analysts can actually work
through. If recall ever matters more — say, before an audit — you lower the threshold
and the curve tells you exactly what you get.

## Slide 14 — Generalisation (1:00 · total 17:15)

How honest are these numbers? Two evaluations. On held-out data from known routes:
PR-AUC zero-point-four-nine — that is close to production reality, where most traffic
repeats. On the separate file of almost entirely new routes: zero-point-three-zero — a
third lower, but far above the floor, so the model is not just memorising.

One ablation matters: features keyed to a
record's statistical group scored zero-point-eight-zero in distribution — and collapsed
to zero-point-zero-five across files, flagging eighty percent of everything. They had
memorised the groups, not the anomalies. They were excluded. And cross-file performance
was still rising when the data ran out — so these numbers are a floor set by data
volume, not a ceiling of the approach.

## Slide 15 — RQ1: How Far Do They Differ? (1:00 · total 18:15)

So, research question one. The two approaches agree on ninety-six-point-six percent of
records. The disagreements are the interesting part, and they split in two. Eight
hundred sixty-five records are flagged only by the model — and ninety-eight percent of
those sit inside what the rules call clean. Those are candidate anomalies in the
majority that no rule ever inspects — the model's potential added value. In the other
direction, four thousand two hundred fifty-four records are flagged only by the rules —
that is the model's low recall, seen from the other side.

In short: the difference is near zero on frequent, rule-shaped patterns — and near
total on the rare ones.

## Slide 16 — RQ2: Does the Model Detect More Than the Rules? (0:45 · total 19:00)

Research question two: does the model detect more than the rules do? It points beyond
them — eight hundred sixty-five flags that no rule raises, and ninety-eight percent of
those sit inside the majority the rules call clean: the space no rule ever inspects.
That is the model's potential added value.

Whether those flags are real anomalies or noise is something the rule labels
structurally cannot tell us — every extra flag is scored as a false alarm by
construction. That is exactly why the blind expert review from the methodology exists;
its results were pending at submission.

What the numbers do settle is that the model cannot stand alone: it misses three in
four anomalies at the precision-first threshold, it cannot learn the scarce patterns
the deterministic rules catch every time, and it weakens on unfamiliar routes. So the
rules stay — and the model's value is what it adds on top of them.

## Slide 17 — Limitations & Future Work (0:30 · total 19:30)

Briefly, the honest boundaries: one export, one airline group, three rules; the rule
labels cap what the model can discover; the metric is one-sided; and the split was
random, not temporal. Future work follows directly: unsupervised detection to escape
the label ceiling, more data and more rules, temporal validation — and completing the
expert review.

## Slide 18 — Conclusion (0:30 · total 20:00)

To conclude: this thesis contributes a hybrid two-stage architecture, the first
empirical rule-versus-ML comparison on the same real tabular flight data, and a working
configurable web application. The one sentence I would like you to take away: rules and
machine learning are not rivals — they are two stages of the same pipeline. The rules
remain the trusted, explainable core; the model adds a layer that surfaces candidates
in the ninety-five percent no rule ever inspects.

Thank you — I look forward to your questions.

---

*Q&A preparation: see `defense_cheatsheet.md` for every number plus answers to the
fifteen most likely questions.*
