---
title: "Hybrid Plausibility Checking of Aviation Flight Data"
subtitle: "A Comparison of Rule-Based and Machine-Learning Approaches to Anomaly Detection"
author: "Bryan Austin Suharta"
date: "Bachelor Kolloquium · Hochschule Darmstadt · July 2026<br><span style='font-size:0.62em;color:#8fa8bd;'>in cooperation with datatactics GmbH &amp; Lufthansa Systems GmbH &nbsp;·&nbsp; Examiners: Prof. Dr. Martin Stiemerling · Prof. Dr. Peter Kling</span>"
theme: moon
transition: slide
highlight-style: breezeDark
progress: true
slideNumber: true
hash: true
navigationMode: linear
css: styles.css
---

### Why Flight-Data Plausibility?

```{=html}
<div style="display:flex;flex-direction:column;gap:0.8em;margin-top:0.5em;font-size:0.62em;">
  <div style="display:flex;gap:0.9em;">
    <div style="flex:1;background:#102238;border:1px solid #2D5A7B;border-radius:12px;padding:0.7em;text-align:center;">
      <div style="font-size:1.9em;font-weight:bold;color:#5BA3C9;">5–8 TB</div>
      <div style="color:#9ab0c4;margin-top:0.2em;">data per flight</div>
    </div>
    <div style="flex:1;background:#102238;border:1px solid #2D5A7B;border-radius:12px;padding:0.7em;text-align:center;">
      <div style="font-size:1.9em;font-weight:bold;color:#5BA3C9;">~98M TB</div>
      <div style="color:#9ab0c4;margin-top:0.2em;">per year by 2026</div>
    </div>
    <div style="flex:1;background:#102238;border:2px solid #F7BB40;border-radius:12px;padding:0.7em;text-align:center;">
      <div style="font-size:1.9em;font-weight:bold;color:#F7BB40;">1 record</div>
      <div style="color:#9ab0c4;margin-top:0.2em;">can distort an audit</div>
    </div>
  </div>
  <div style="display:flex;align-items:stretch;gap:0.5em;background:#0a1828;border-radius:12px;padding:0.6em 0.9em;color:#FBF9F5;text-align:center;">
    <div style="flex:1.1;display:flex;flex-direction:column;justify-content:center;">
      <div>Netline/Ops · FDDC<br>airline feeds</div>
      <div style="color:#8fa8bd;margin-top:0.15em;">raw XML</div>
    </div>
    <div style="display:flex;align-items:center;color:#F7BB40;font-size:1.3em;">→</div>
    <div style="flex:1.3;display:flex;flex-direction:column;justify-content:center;background:#102238;border:1px solid #2D5A7B;border-radius:10px;padding:0.4em;">
      <div style="color:#5BA3C9;font-weight:bold;font-size:1.1em;">FLT — the ETL pipeline</div>
      <div style="color:#8fa8bd;margin-top:0.15em;">by datatactics: parse · unify · harmonise</div>
    </div>
    <div style="display:flex;align-items:center;color:#F7BB40;font-size:1.3em;">→</div>
    <div style="flex:1.1;display:flex;flex-direction:column;justify-content:center;">
      <div>Fuel Data Warehouse</div>
      <div style="color:#8fa8bd;margin-top:0.15em;">ETS audits · efficiency analysis</div>
    </div>
  </div>
  <div>
    <div style="text-align:center;color:#8fa8bd;margin-bottom:0.35em;">where implausible values come from</div>
    <div style="display:flex;gap:0.5em;text-align:center;color:#ccc;">
      <div style="flex:1;background:#1a1a2e;border-radius:8px;padding:0.45em 0.3em;">missing<br>measurements</div>
      <div style="flex:1;background:#1a1a2e;border-radius:8px;padding:0.45em 0.3em;">mis-calibrated<br>equipment</div>
      <div style="flex:1;background:#1a1a2e;border-radius:8px;padding:0.45em 0.3em;">differing calculation<br>methods</div>
      <div style="flex:1;background:#1a1a2e;border-radius:8px;padding:0.45em 0.3em;">manual-entry<br>typos</div>
      <div style="flex:1;background:#1a1a2e;border-radius:8px;padding:0.45em 0.3em;">deliberate<br>misreporting</div>
    </div>
  </div>
  <div style="text-align:center;font-size:1.2em;color:#FBF9F5;margin-top:0.15em;">
    Plausibility — <span style="color:#5BA3C9;font-weight:bold;">is this value credible in its context?</span>
  </div>
</div>
```

---

### Current Practice: Hand-Written Rules

```{=html}
<div style="display:flex;flex-direction:column;gap:1em;margin-top:1em;font-size:0.7em;align-items:center;">
  <div style="color:#9ab0c4;">today: hand-written Groovy rules — fuel columns only</div>
  <div style="font-family:monospace;font-size:1.3em;color:#9ecae1;background:#1a1a2e;border-radius:10px;padding:0.6em 1.2em;">
    |block − (remaining + uplift)| ≤ 200 kg
  </div>
  <div style="display:flex;gap:0.9em;width:100%;">
    <div style="flex:1;background:#2a1a1a;border:1px solid #6a3a3a;border-radius:10px;padding:0.7em;text-align:center;color:#ff6b6b;font-weight:bold;font-size:1.1em;">Brittle</div>
    <div style="flex:1;background:#2a1a1a;border:1px solid #6a3a3a;border-radius:10px;padding:0.7em;text-align:center;color:#ff6b6b;font-weight:bold;font-size:1.1em;">Expensive</div>
    <div style="flex:1;background:#2a1a1a;border:1px solid #6a3a3a;border-radius:10px;padding:0.7em;text-align:center;color:#ff6b6b;font-weight:bold;font-size:1.1em;">Incomplete</div>
  </div>
  <div style="color:#FBF9F5;margin-top:0.4em;font-size:1.05em;">
    10,000 kg trip fuel: a <em>valid</em> number — <span style="color:#F7BB40;font-weight:bold;">implausible</span> on a 30-minute leg
  </div>
</div>
```

---

### Research Questions

```{=html}
<div style="display:flex;flex-direction:column;gap:1em;margin-top:1.6em;font-size:0.85em;">
  <div style="display:flex;gap:1em;align-items:stretch;">
    <div style="flex:0 0 100px;display:flex;align-items:center;justify-content:center;background:#102238;border:2px solid #5BA3C9;border-radius:12px;color:#5BA3C9;font-weight:bold;font-size:1.3em;">RQ1</div>
    <div style="flex:1;display:flex;align-items:center;background:#1a1a2e;border-radius:12px;padding:0.8em 1.2em;">
      <div style="color:#FBF9F5;line-height:1.5;">How far do <span style="color:#5BA3C9;font-weight:bold;">rule-based</span> and <span style="color:#10B981;font-weight:bold;">ML-based</span> verdicts differ on the same flight data?</div>
    </div>
  </div>
  <div style="display:flex;gap:1em;align-items:stretch;">
    <div style="flex:0 0 100px;display:flex;align-items:center;justify-content:center;background:#102238;border:2px solid #F7BB40;border-radius:12px;color:#F7BB40;font-weight:bold;font-size:1.3em;">RQ2</div>
    <div style="flex:1;display:flex;align-items:center;background:#1a1a2e;border-radius:12px;padding:0.8em 1.2em;">
      <div style="color:#FBF9F5;line-height:1.5;">Can ML <span style="color:#F7BB40;font-weight:bold;">replace</span> the rules — or only <span style="color:#F7BB40;font-weight:bold;">complement</span> them?</div>
    </div>
  </div>
</div>
```

---

### Methodology

```{=html}
<div style="display:flex;flex-direction:column;gap:0.9em;margin-top:0.7em;font-size:0.62em;">
  <div>
    <div style="text-align:center;color:#8fa8bd;margin-bottom:0.4em;">Design Science Research — iterative, with Lufthansa Systems</div>
    <div style="display:flex;gap:0.5em;align-items:center;text-align:center;">
      <div style="flex:1;background:#102238;border:1px solid #2D5A7B;border-radius:10px;padding:0.7em;color:#FBF9F5;">real industrial<br>problem</div>
      <div style="color:#F7BB40;font-size:1.4em;">→</div>
      <div style="flex:1;background:#102238;border:1px solid #2D5A7B;border-radius:10px;padding:0.7em;color:#FBF9F5;">build the artefact<br><span style="color:#8fa8bd;">Streamlit app · rules editable at runtime</span></div>
      <div style="color:#F7BB40;font-size:1.4em;">→</div>
      <div style="flex:1;background:#102238;border:1px solid #2D5A7B;border-radius:10px;padding:0.7em;color:#FBF9F5;">evaluate<br>empirically</div>
    </div>
  </div>
  <div style="display:flex;gap:0.9em;text-align:center;">
    <div style="flex:1;background:#0d1a0d;border:1px solid #3a6a4a;border-radius:12px;padding:0.8em;">
      <div style="color:#10B981;font-weight:bold;font-size:1.1em;">Numerical</div>
      <div style="color:#9ab0c4;margin-top:0.35em;">PR-AUC vs no-skill floor<br>held-out + unseen-routes file</div>
    </div>
    <div style="flex:1;background:#1a1208;border:1px solid #6a5a2a;border-radius:12px;padding:0.8em;">
      <div style="color:#F7BB40;font-weight:bold;font-size:1.1em;">Manual</div>
      <div style="color:#9ab0c4;margin-top:0.35em;">domain experts judge<br>the disagreements — blind</div>
    </div>
  </div>
  <div>
    <div style="text-align:center;color:#8fa8bd;margin-bottom:0.4em;">how the expert check works</div>
    <div style="display:flex;gap:0.5em;text-align:center;color:#ccc;">
      <div style="flex:1;background:#1a1a2e;border-radius:8px;padding:0.5em 0.3em;">one flight<br>per row</div>
      <div style="flex:1.3;background:#1a1a2e;border-radius:8px;padding:0.5em 0.3em;">rule vs Model 1 vs Model 2<br>— anonymised</div>
      <div style="flex:1;background:#1a1a2e;border-radius:8px;padding:0.5em 0.3em;">IQR bounds<br>as evidence</div>
      <div style="flex:1;background:#1a1a2e;border-radius:8px;padding:0.5em 0.3em;">tick the correct<br>verdicts</div>
    </div>
  </div>
</div>
```

---

### Approach: A Hybrid Two-Stage Pipeline

```{=html}
<div style="display:flex;gap:0.8em;margin-top:1.6em;font-size:0.66em;align-items:stretch;">
  <div style="flex:0 0 16%;display:flex;flex-direction:column;justify-content:center;background:#0a1828;border:1px solid #2D5A7B;border-radius:12px;padding:0.8em;text-align:center;">
    <div style="color:#FBF9F5;font-weight:bold;">Flight data</div>
    <div style="color:#8fa8bd;margin-top:0.3em;">~1.4M legs</div>
  </div>
  <div style="display:flex;align-items:center;color:#F7BB40;font-size:1.6em;">→</div>
  <div style="flex:1;background:#102238;border:2px solid #5BA3C9;border-radius:12px;padding:0.9em 1em;text-align:center;">
    <div style="color:#5BA3C9;font-weight:bold;text-transform:uppercase;letter-spacing:0.05em;">Stage 1</div>
    <div style="color:#FBF9F5;margin-top:0.5em;line-height:1.6;">statistical detector<br>+ rule engine</div>
    <div style="margin-top:0.6em;color:#9ab0c4;">labels every record</div>
    <div style="margin-top:0.3em;"><span style="color:#10B981;">clean</span> · <span style="color:#F7BB40;">explained</span> · <span style="color:#ff6b6b;font-weight:bold;">anomaly</span></div>
  </div>
  <div style="display:flex;align-items:center;color:#F7BB40;font-size:1.6em;">→</div>
  <div style="flex:1;background:#102238;border:2px solid #10B981;border-radius:12px;padding:0.9em 1em;text-align:center;">
    <div style="color:#10B981;font-weight:bold;text-transform:uppercase;letter-spacing:0.05em;">Stage 2</div>
    <div style="color:#FBF9F5;margin-top:0.5em;line-height:1.6;">classifier trained<br>on Stage 1's labels</div>
    <div style="margin-top:0.6em;color:#9ab0c4;">LightGBM · Logistic Regression</div>
  </div>
</div>
<div style="text-align:center;margin-top:1.2em;font-size:0.75em;color:#FBF9F5;">
  the rules <span style="color:#F7BB40;font-weight:bold;">generate the training signal</span> for the model
</div>
```

---

### Stage 1 — How a Record Gets Its Label

```{=html}
<div style="display:flex;flex-direction:column;gap:0.7em;margin-top:1em;font-size:0.66em;">
  <div style="text-align:center;color:#9ab0c4;">
    per cluster (airline · year · aircraft type · route) — flag values outside
    <span style="font-family:monospace;color:#9ecae1;">[Q1 − 1.5·IQR, &nbsp;Q3 + 1.5·IQR]</span>
  </div>
  <div style="background:#0d1a0d;border:1px solid #3a6a4a;border-radius:10px;padding:0.6em 1em;display:flex;justify-content:space-between;align-items:center;">
    <span style="color:#FBF9F5;">Revenue flight · TOW <strong>57,547</strong> — inside bounds</span>
    <span style="background:#1a3a2a;border:1px solid #3a6a4a;color:#10B981;padding:3px 14px;border-radius:20px;font-weight:bold;">clean</span>
  </div>
  <div style="background:#1a1208;border:1px solid #6a5a2a;border-radius:10px;padding:0.6em 1em;display:flex;justify-content:space-between;align-items:center;">
    <span style="color:#FBF9F5;">Ferry flight · TOW <strong>48,115</strong> — flagged, ferry rule fires</span>
    <span style="background:#3a2a1a;border:1px solid #7a5a2a;color:#F7BB40;padding:3px 14px;border-radius:20px;font-weight:bold;">explained</span>
  </div>
  <div style="background:#2a1a1a;border:1px solid #6a3a3a;border-radius:10px;padding:0.6em 1em;display:flex;justify-content:space-between;align-items:center;">
    <span style="color:#FBF9F5;">Revenue flight · TOW <strong>50,805</strong> — flagged, <strong>no rule fires</strong></span>
    <span style="background:#3a1a1a;border:1px solid #7a3a3a;color:#ff6b6b;padding:3px 14px;border-radius:20px;font-weight:bold;">anomaly</span>
  </div>
  <div style="text-align:center;color:#FBF9F5;margin-top:0.5em;font-size:1.05em;">
    unexplained anomalies = <span style="color:#F7BB40;font-weight:bold;">the detection target</span>
  </div>
</div>
```

---

### Data & Ground Truth

```{=html}
<div style="display:flex;flex-direction:column;gap:0.8em;margin-top:0.9em;font-size:0.66em;">
  <div style="display:flex;gap:0.9em;text-align:center;">
    <div style="flex:1;background:#102238;border:1px solid #2D5A7B;border-radius:10px;padding:0.55em;"><strong style="font-size:1.25em;">~1.4M legs</strong><br><span style="color:#8fa8bd;">one 3 GB export</span></div>
    <div style="flex:1;background:#102238;border:1px solid #2D5A7B;border-radius:10px;padding:0.55em;"><strong style="font-size:1.25em;">654,053</strong><br><span style="color:#8fa8bd;">labelled records</span></div>
    <div style="flex:1;background:#102238;border:1px solid #2D5A7B;border-radius:10px;padding:0.55em;"><strong style="font-size:1.25em;">149,176</strong><br><span style="color:#8fa8bd;">unseen-routes test</span></div>
  </div>
  <div style="background:#1a1a2e;border-radius:10px;padding:0.55em 1em;display:flex;justify-content:space-between;align-items:center;">
    <span style="color:#FBF9F5;">clean</span><span style="color:#FBF9F5;"><strong>621,286</strong> · 94.99%</span>
  </div>
  <table style="width:100%;border-collapse:collapse;color:#FBF9F5;">
    <tbody>
      <tr>
        <td style="padding:4px 8px;width:30%;text-align:left;color:#ff6b6b;font-weight:bold;white-space:nowrap;">anomaly — the target</td>
        <td style="padding:4px 0;"><div style="background:#BA8820;height:22px;width:100%;border-radius:0 5px 5px 0;"></div></td>
        <td style="padding:4px 10px;width:22%;text-align:right;white-space:nowrap;"><strong>17,602</strong> · 2.69%</td>
      </tr>
      <tr>
        <td style="padding:4px 8px;text-align:left;white-space:nowrap;">Low Cargo</td>
        <td style="padding:4px 0;"><div style="background:#3E8FC6;height:22px;width:76.7%;border-radius:0 5px 5px 0;"></div></td>
        <td style="padding:4px 10px;text-align:right;white-space:nowrap;">13,503 · 2.06%</td>
      </tr>
      <tr>
        <td style="padding:4px 8px;text-align:left;white-space:nowrap;">Ferry flight</td>
        <td style="padding:4px 0;"><div style="background:#3E8FC6;height:22px;width:8%;border-radius:0 5px 5px 0;"></div></td>
        <td style="padding:4px 10px;text-align:right;white-space:nowrap;">1,403 · 0.21%</td>
      </tr>
      <tr>
        <td style="padding:4px 8px;text-align:left;white-space:nowrap;">ZFW OK – No Uplift</td>
        <td style="padding:4px 0;"><div style="background:#3E8FC6;height:22px;width:1.5%;min-width:3px;border-radius:0 5px 5px 0;"></div></td>
        <td style="padding:4px 10px;text-align:right;white-space:nowrap;">259 · 0.04%</td>
      </tr>
    </tbody>
  </table>
  <div style="text-align:center;color:#8fa8bd;">bars scaled to the anomaly class</div>
</div>
```

---

### Stage 2 — Learning Setup

```{=html}
<div style="display:flex;flex-direction:column;gap:1em;margin-top:1.4em;font-size:0.7em;">
  <div style="text-align:center;color:#9ab0c4;">multiclass — 5 classes · target: <span style="color:#ff6b6b;font-weight:bold;">anomaly</span></div>
  <div style="display:flex;gap:0.9em;text-align:center;">
    <div style="flex:1;background:#102238;border:1px solid #4a4a5a;border-radius:12px;padding:0.9em;">
      <div style="color:#aaa;font-weight:bold;font-size:1.1em;">Dummy</div>
      <div style="color:#8fa8bd;margin-top:0.35em;">the no-skill floor</div>
    </div>
    <div style="flex:1;background:#102238;border:1px solid #3a5a8a;border-radius:12px;padding:0.9em;">
      <div style="color:#5BA3C9;font-weight:bold;font-size:1.1em;">Logistic Regression</div>
      <div style="color:#8fa8bd;margin-top:0.35em;">linear baseline</div>
    </div>
    <div style="flex:1;background:#102238;border:2px solid #10B981;border-radius:12px;padding:0.9em;">
      <div style="color:#10B981;font-weight:bold;font-size:1.1em;">LightGBM</div>
      <div style="color:#8fa8bd;margin-top:0.35em;">gradient-boosted trees</div>
    </div>
  </div>
  <div style="text-align:center;color:#FBF9F5;font-size:1.05em;margin-top:0.4em;">
    ~200 raw columns → <span style="color:#F7BB40;font-weight:bold;">100 features</span> · 50 domain-engineered
  </div>
</div>
```

---

### Training for Rare Anomalies

```{=html}
<div style="display:flex;flex-direction:column;gap:0.9em;margin-top:1.4em;font-size:0.7em;">
  <div style="display:flex;gap:0.9em;text-align:center;">
    <div style="flex:1;background:#102238;border:1px solid #2D5A7B;border-radius:12px;padding:0.9em;">
      <div style="color:#5BA3C9;font-weight:bold;font-size:1.15em;">Undersampling</div>
      <div style="color:#8fa8bd;margin-top:0.35em;">clean trimmed to 10 : 1</div>
    </div>
    <div style="flex:1;background:#102238;border:1px solid #2D5A7B;border-radius:12px;padding:0.9em;">
      <div style="color:#5BA3C9;font-weight:bold;font-size:1.15em;">Optuna tuning</div>
      <div style="color:#8fa8bd;margin-top:0.35em;">450 trees · depth 10</div>
    </div>
    <div style="flex:1;background:#0d1a0d;border:2px solid #10B981;border-radius:12px;padding:0.9em;">
      <div style="color:#10B981;font-weight:bold;font-size:1.15em;">Threshold 0.9</div>
      <div style="color:#8fa8bd;margin-top:0.35em;">precision first</div>
    </div>
  </div>
  <div style="text-align:center;color:#FBF9F5;font-size:1.1em;margin-top:0.5em;">
    miss → <span style="color:#10B981;font-weight:bold;">cheap</span> &nbsp;·&nbsp; false alarm → <span style="color:#ff6b6b;font-weight:bold;">expensive</span>
  </div>
  <div style="text-align:center;color:#8fa8bd;">metric: PR-AUC · no-skill floor 0.027</div>
</div>
```

---

### Results — Model Comparison, Step by Step

<iframe scrolling="no" style="border:none;" seamless="seamless"
  data-src="assets/model_buildup.html" height="615" width="100%"></iframe>

---

### The Final Model

```{=html}
<div style="display:flex;gap:1em;margin-top:0.9em;font-size:0.66em;align-items:flex-start;">
  <div style="flex:1.3;">
    <table style="width:100%;border-collapse:collapse;text-align:center;color:#FBF9F5;">
      <thead>
        <tr style="color:#8fa8bd;font-size:0.9em;text-transform:uppercase;letter-spacing:0.04em;border-bottom:1px solid #333;">
          <th style="padding:6px;text-align:left;">Class</th><th style="padding:6px;">P</th><th style="padding:6px;">R</th><th style="padding:6px;">F1</th>
        </tr>
      </thead>
      <tbody>
        <tr style="border-bottom:1px solid #2a2a3a;"><td style="padding:6px;text-align:left;">clean</td><td>0.97</td><td>0.99</td><td>0.98</td></tr>
        <tr style="border-bottom:1px solid #2a2a3a;"><td style="padding:6px;text-align:left;">Low Cargo</td><td>0.61</td><td>0.75</td><td>0.67</td></tr>
        <tr style="border-bottom:1px solid #2a2a3a;"><td style="padding:6px;text-align:left;">Ferry flight</td><td>0.57</td><td>0.81</td><td>0.67</td></tr>
        <tr style="border-bottom:1px solid #2a2a3a;color:#ff6b6b;"><td style="padding:6px;text-align:left;">ZFW OK – No Uplift</td><td>0.06</td><td>0.03</td><td><strong>0.04</strong></td></tr>
        <tr style="background:#102238;"><td style="padding:7px 6px;text-align:left;font-weight:bold;color:#F7BB40;">anomaly</td>
          <td style="color:#F7BB40;font-weight:bold;">0.76</td><td style="color:#F7BB40;font-weight:bold;">0.24</td><td style="color:#F7BB40;font-weight:bold;">0.36</td></tr>
      </tbody>
    </table>
    <div style="text-align:center;color:#FBF9F5;margin-top:0.8em;font-size:1.05em;">
      flags right <span style="color:#10B981;font-weight:bold;">3× out of 4</span> · finds <span style="color:#ff6b6b;font-weight:bold;">1 in 4</span>
    </div>
  </div>
  <div style="flex:1;display:flex;flex-direction:column;gap:0.6em;">
    <div style="display:grid;grid-template-columns:96px 1fr 1fr;gap:2px;text-align:center;color:#FBF9F5;">
      <div></div>
      <div style="color:#8fa8bd;padding:4px;">pred. anomaly</div>
      <div style="color:#8fa8bd;padding:4px;">pred. not</div>
      <div style="color:#8fa8bd;display:flex;align-items:center;justify-content:flex-end;padding-right:8px;">anomaly</div>
      <div style="background:rgba(62,143,198,0.45);border-radius:8px 0 0 0;padding:0.9em 0;"><strong>1,252</strong></div>
      <div style="background:rgba(224,82,82,0.30);border-radius:0 8px 0 0;padding:0.9em 0;"><strong>4,028</strong></div>
      <div style="color:#8fa8bd;display:flex;align-items:center;justify-content:flex-end;padding-right:8px;">not</div>
      <div style="background:rgba(224,82,82,0.30);border-radius:0 0 0 8px;padding:0.9em 0;"><strong>397</strong></div>
      <div style="background:rgba(62,143,198,0.45);border-radius:0 0 8px 0;padding:0.9em 0;"><strong>190,539</strong></div>
    </div>
    <div style="text-align:center;color:#8fa8bd;margin-top:0.4em;">errors ≈ anomalies predicted clean</div>
  </div>
</div>
```

---

### Choosing the Operating Point

```{=html}
<div style="display:flex;gap:1em;margin-top:0.8em;font-size:0.7em;align-items:center;">
  <div style="flex:1.7;">
    <svg viewBox="0 0 1000 360" style="width:100%;height:auto;" role="img" aria-label="Line chart: as the decision threshold rises from 0.3 to 0.9, precision rises from 0.46 to 0.76 while recall falls from 0.53 to 0.24; F1 declines gently to 0.36.">
      <g stroke="#24435f" stroke-width="1">
        <line x1="60" y1="300" x2="780" y2="300"/>
        <line x1="60" y1="230" x2="780" y2="230"/>
        <line x1="60" y1="160" x2="780" y2="160"/>
        <line x1="60" y1="90"  x2="780" y2="90"/>
        <line x1="60" y1="20"  x2="780" y2="20"/>
      </g>
      <g fill="#7d93a8" font-size="17" text-anchor="end">
        <text x="50" y="305">0</text>
        <text x="50" y="235">0.2</text>
        <text x="50" y="165">0.4</text>
        <text x="50" y="95">0.6</text>
        <text x="50" y="25">0.8</text>
      </g>
      <g fill="#7d93a8" font-size="17" text-anchor="middle">
        <text x="60"  y="326">0.3</text>
        <text x="180" y="326">0.4</text>
        <text x="300" y="326">0.5</text>
        <text x="420" y="326">0.6</text>
        <text x="540" y="326">0.7</text>
        <text x="660" y="326">0.8</text>
        <text x="780" y="326">0.9</text>
        <text x="420" y="352" fill="#9ab0c4">decision threshold</text>
      </g>
      <line x1="780" y1="20" x2="780" y2="300" stroke="#F7BB40" stroke-width="1.5" stroke-dasharray="6 5"/>
      <text x="770" y="14" fill="#F7BB40" font-size="16" text-anchor="end">chosen operating point</text>
      <polyline points="60,128.5 180,126.8 300,129.6 420,133.4 540,141.1 660,152 780,173.7" fill="none" stroke="#8CA3B8" stroke-width="2.5" stroke-dasharray="3 5"/>
      <polyline points="60,114.9 180,129.9 300,145.7 420,160.4 540,176.1 660,192.9 780,217.1" fill="none" stroke="#BA8820" stroke-width="3"/>
      <polyline points="60,140.1 180,122.9 300,109.3 420,93.5 540,78.1 660,59.2 780,34.4" fill="none" stroke="#3E8FC6" stroke-width="3"/>
      <g fill="#BA8820" stroke="#0E2A42" stroke-width="2">
        <circle cx="60" cy="114.9" r="4.5"/><circle cx="180" cy="129.9" r="4.5"/><circle cx="300" cy="145.7" r="4.5"/><circle cx="420" cy="160.4" r="4.5"/><circle cx="540" cy="176.1" r="4.5"/><circle cx="660" cy="192.9" r="4.5"/><circle cx="780" cy="217.1" r="6"/>
      </g>
      <g fill="#3E8FC6" stroke="#0E2A42" stroke-width="2">
        <circle cx="60" cy="140.1" r="4.5"/><circle cx="180" cy="122.9" r="4.5"/><circle cx="300" cy="109.3" r="4.5"/><circle cx="420" cy="93.5" r="4.5"/><circle cx="540" cy="78.1" r="4.5"/><circle cx="660" cy="59.2" r="4.5"/><circle cx="780" cy="34.4" r="6"/>
      </g>
      <g font-size="19" font-weight="bold">
        <text x="792" y="40" fill="#E8EFF5">Precision 0.76</text>
        <text x="792" y="179" fill="#9ab0c4">F1 0.36</text>
        <text x="792" y="223" fill="#E8EFF5">Recall 0.24</text>
      </g>
    </svg>
  </div>
  <div style="flex:1;display:flex;flex-direction:column;gap:0.8em;text-align:center;">
    <div style="background:#102238;border:1px solid #2D5A7B;border-radius:12px;padding:0.9em;color:#FBF9F5;font-size:1.1em;">
      at 0.9 → <span style="color:#10B981;font-weight:bold;">¾ of flags are real</span>
    </div>
    <div style="background:#1a1a2e;border-radius:12px;padding:0.9em;color:#8fa8bd;">
      movable later —<br>no retraining needed
    </div>
  </div>
</div>
```

---

### Generalisation — the Honest Range

```{=html}
<div style="display:flex;flex-direction:column;gap:1em;margin-top:1.2em;font-size:0.7em;">
  <div style="display:flex;gap:1em;">
    <div style="flex:1;background:#0d1a0d;border:2px solid #10B981;border-radius:12px;padding:1em;text-align:center;">
      <div style="color:#10B981;font-weight:bold;text-transform:uppercase;letter-spacing:0.05em;">known routes</div>
      <div style="font-size:2em;font-weight:bold;color:#FBF9F5;margin-top:0.2em;">0.49</div>
      <div style="color:#8fa8bd;margin-top:0.2em;">P 0.76 · R 0.24</div>
    </div>
    <div style="flex:1;background:#1a1208;border:2px solid #F7BB40;border-radius:12px;padding:1em;text-align:center;">
      <div style="color:#F7BB40;font-weight:bold;text-transform:uppercase;letter-spacing:0.05em;">new routes</div>
      <div style="font-size:2em;font-weight:bold;color:#FBF9F5;margin-top:0.2em;">0.30</div>
      <div style="color:#8fa8bd;margin-top:0.2em;">P 0.52 · R 0.18</div>
    </div>
  </div>
  <div style="text-align:center;color:#ff6b6b;font-size:1.05em;">
    group-keyed features: 0.80 in-distribution → <strong>0.05</strong> across files — excluded
  </div>
  <div style="text-align:center;color:#8fa8bd;">still rising with more data — a floor, not a ceiling</div>
</div>
```

---

### RQ1 — How Far Do They Differ?

```{=html}
<div style="display:flex;flex-direction:column;gap:1em;margin-top:1.2em;font-size:0.7em;align-items:center;">
  <div style="font-size:2.4em;font-weight:bold;color:#10B981;">96.6% agreement</div>
  <div style="display:flex;gap:1em;width:100%;text-align:center;">
    <div style="flex:1;background:#102238;border:1px solid #2D5A7B;border-radius:12px;padding:0.9em;">
      <div style="color:#5BA3C9;font-weight:bold;font-size:1.3em;">865 model-only flags</div>
      <div style="color:#8fa8bd;margin-top:0.35em;">98% land on records the rules call clean</div>
    </div>
    <div style="flex:1;background:#102238;border:1px solid #2D5A7B;border-radius:12px;padding:0.9em;">
      <div style="color:#ff6b6b;font-weight:bold;font-size:1.3em;">4,254 rule-only flags</div>
      <div style="color:#8fa8bd;margin-top:0.35em;">the model's low recall, seen from the other side</div>
    </div>
  </div>
  <div style="text-align:center;color:#FBF9F5;font-size:1.1em;margin-top:0.4em;">
    near-zero difference on <span style="color:#10B981;">frequent</span> patterns · near-total on <span style="color:#ff6b6b;">rare</span> ones
  </div>
</div>
```

---

### RQ2 — Replace or Complement?

```{=html}
<div style="display:flex;flex-direction:column;gap:0.9em;margin-top:1em;font-size:0.7em;">
  <div style="text-align:center;background:#102238;border:2px solid #F7BB40;border-radius:12px;padding:0.7em;color:#FBF9F5;font-size:1.25em;">
    <span style="color:#F7BB40;font-weight:bold;">Complement</span> — replacement is not supported
  </div>
  <div style="display:flex;gap:0.8em;text-align:center;">
    <div style="flex:1;background:#1a1a2e;border-radius:10px;padding:0.7em;color:#ccc;">trained on<br>rule labels</div>
    <div style="flex:1;background:#1a1a2e;border-radius:10px;padding:0.7em;color:#ccc;">recall<br>24%</div>
    <div style="flex:1;background:#1a1a2e;border-radius:10px;padding:0.7em;color:#ccc;">scarce classes<br>unlearnable</div>
    <div style="flex:1;background:#1a1a2e;border-radius:10px;padding:0.7em;color:#ccc;">0.49 → 0.30<br>off-route</div>
  </div>
  <div style="text-align:center;color:#FBF9F5;font-size:1.05em;">
    …and the rules can't do <span style="color:#5BA3C9;">soft multivariate cases</span> either → <span style="color:#10B981;font-weight:bold;">keep both</span>
  </div>
  <div style="text-align:center;color:#8fa8bd;">decisive test: blind expert review at Lufthansa Systems — pending</div>
</div>
```

---

### Limitations & Future Work

```{=html}
<div style="display:flex;gap:1em;margin-top:1.2em;font-size:0.7em;">
  <div style="flex:1;background:#2a1a1a;border:1px solid #6a3a3a;border-radius:12px;padding:1em 1.3em;">
    <div style="color:#ff6b6b;font-weight:bold;text-transform:uppercase;letter-spacing:0.05em;margin-bottom:0.6em;">Limitations</div>
    <ul style="color:#ccc;line-height:1.9;margin:0;padding-left:1.1em;">
      <li>one export · one airline group</li>
      <li>rule labels cap the model</li>
      <li>one-sided metric</li>
      <li>no temporal split</li>
    </ul>
  </div>
  <div style="flex:1;background:#0d1a0d;border:1px solid #3a6a4a;border-radius:12px;padding:1em 1.3em;">
    <div style="color:#10B981;font-weight:bold;text-transform:uppercase;letter-spacing:0.05em;margin-bottom:0.6em;">Future work</div>
    <ul style="color:#ccc;line-height:1.9;margin:0;padding-left:1.1em;">
      <li>unsupervised detection</li>
      <li>more data · more rules</li>
      <li>temporal validation</li>
      <li>finish the expert review</li>
    </ul>
  </div>
</div>
```

---

### Conclusion

```{=html}
<div style="display:flex;flex-direction:column;gap:1.1em;margin-top:1.1em;font-size:0.7em;">
  <div style="display:flex;gap:0.9em;text-align:center;">
    <div style="flex:1;background:#102238;border:1px solid #2D5A7B;border-radius:12px;padding:0.8em;color:#10B981;font-weight:bold;font-size:1.05em;">Hybrid architecture</div>
    <div style="flex:1;background:#102238;border:1px solid #2D5A7B;border-radius:12px;padding:0.8em;color:#10B981;font-weight:bold;font-size:1.05em;">Empirical comparison</div>
    <div style="flex:1;background:#102238;border:1px solid #2D5A7B;border-radius:12px;padding:0.8em;color:#10B981;font-weight:bold;font-size:1.05em;">Working web app</div>
  </div>
  <div style="text-align:center;background:#1a1a2e;border-radius:14px;padding:1em;">
    <div style="font-size:1.5em;font-weight:bold;color:#FBF9F5;line-height:1.45;">
      Rules and machine learning are not rivals —<br>
      <span style="color:#5BA3C9;">two stages of the same pipeline.</span>
    </div>
  </div>
  <div style="text-align:center;color:#F7BB40;font-size:1.15em;">Thank you.</div>
</div>
```
