---
title: "Regime-Preserving Bootstrap for FX Fair-Value Models"
date: 2026-06-17
author: ["David Vossebürger"]
description: "A bootstrap methodology that preserves exchange-rate regime structure when evaluating fair-value models of FX."
summary: "Two regime-aware bootstrap resamplers (Regime-Uniform and Regime-Markov) for FX fair-value strategies (BEER, FEER, PPP), validated against naive baselines on three currency pairs with B = 1999 replicates."

---

<a class="repo-card" href="https://github.com/DavidVossebuerger/MC-Regime" target="_blank" rel="noopener">
  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16" width="20" height="20" fill="currentColor"><path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0 0 16 8c0-4.42-3.58-8-8-8z"/></svg>
  <div class="repo-meta">
    <span class="repo-name">DavidVossebuerger/MC-Regime</span>
    <span class="repo-action">View source code on GitHub →</span>
  </div>
</a>

{{< ssrn id="6890880" >}}

<div class="audit-banner">

**AI usage disclosure.** I do not claim to understand every individual step of the mathematical elaboration in this paper. The foundational idea (regime-aware bootstrap) and the framing of the question (do standard block-bootstrap methods destroy regime structure?) are mine. The mathematical formulation, the implementation, the testing, and the writing of the paper itself were done by large language models (MiniMax M3 and M2.7). I verified the code runs, the numbers reproduce, and the conclusions follow from the data. I do not claim to have hand-derived any of the underlying statistics myself.

</div>

---

### Abstract

This paper documents a regime-preserving bootstrap procedure for validating the out-of-sample robustness of FX fair-value strategies (BEER, FEER, PPP). The point: standard bootstrap methods (IID, Moving Block, Stationary Block) fail to preserve the regime structure of macro-FX data and inflate Sharpe ratio inference (4.6× overstatement on the IID benchmark).

We propose two regime-aware resamplers — Regime-Uniform (block resampling within macro-regimes) and Regime-Markov (block resampling with calibrated Markov transition matrix) — and validate them using five structure metrics, a Sharpe loss metric, an arrangement distance, a stationarity gate per Section 11.7, and Wilcoxon paired tests with Benjamini-Hochberg FDR.

Applied to a BEER pricer for three currency pairs (EURUSD, GBPUSD, USDJPY) with B = 1999 bootstrap replicates, **Regime-Markov** reduces bootstrap Sharpe loss by 1.3× vs. Regime-Uniform and 4.6× vs. IID. The Markov transition structure provides the additional gain over pure block resampling.

The calibrated transition matrix passes the Section 11.7 stationarity gate (TV drift = 0.10), validating forward projections over short horizons of ∼3–5 Markov steps. The paper documents the complete metric definitions, the implementation, and the limits of cross-pair generalization.

*Disclosure: the full AI-attribution statement is at the top of the page.*

---

### Honest framing

The L0 headline rests on three independent currency pairs, so the Wilcoxon pair-level p-value bottoms out at 0.25 (best possible for n = 3). The decision rule is therefore binary, not significance-based: does Regime-Markov beat IID, MBB, SB, and Regime-Uniform simultaneously on the cross-pool L0? Answer: yes on all four. The p-value does not enter the GO decision.

---

### Bootstrap Sharpe loss (cross-pool L0, lower = better)

<figure class="chart-figure">
<p class="chart-title">Cross-pool L0 loss by resampler</p>
<div class="chart-canvas-wrap short"><canvas id="mc-loss-chart"></canvas></div>
<p class="chart-caption">Lower is better. Regime-Markov (0.22) beats IID (1.01) by 4.6× and Regime-Uniform (0.30) by 1.3× on the cross-pool mean of three independent pairs.</p>
</figure>

### Per-pair BEER in-sample Sharpe (2003–2026)

<figure class="chart-figure">
<p class="chart-title">Per-pair Sharpe, signed</p>
<div class="chart-canvas-wrap short"><canvas id="mc-pair-chart"></canvas></div>
<p class="chart-caption">GBPUSD is negative (−0.107) — a documented limitation, not a pipeline bug. The L0 loss for GBPUSD is therefore inflated by construction (the bootstrap has more difficulty reconstructing a negative Sharpe), which is why the cross-pool mean is what gets reported.</p>
</figure>

### Structure losses across five metrics

| Method | L0 (Sharpe) | L1 (corr) | L2 (Wasserstein) | L3_fro | L3_spec | L4_temporal |
|---|---:|---:|---:|---:|---:|---:|
| IID | 1.0146 | 0.0957 | 0.0957 | 0.5636 | 0.2823 | 0.3289 |
| MBB | 0.4324 | 0.1275 | 0.1275 | 0.7231 | 0.3534 | 0.1225 |
| SB | 0.4659 | 0.1202 | 0.1202 | 0.6821 | 0.3337 | 0.1523 |
| Regime-Uniform | 0.2977 | 0.1324 | 0.1207 | 0.6953 | 0.3223 | 0.1195 |
| Regime-Markov | 0.2211 | 0.0839 | 0.1207 | 0.5502 | 0.2433 | 0.0641 |

L1 and L2 are noted in the paper as "discriminates poorly" across methods — joint distribution preservation is a structural property that all block-level resamplers fail on equally. L3_spec (spectral) and L4_temporal show Regime-Markov's structural edge most cleanly.

---

### §11.7 stationarity gate — cap sensitivity

| Cap | TV drift | Gate (TV &lt; 0.10) | Max forbidden mass |
|---:|---:|:---:|---:|
| 0.5 % | 0.0996 | PASS | 0.0 % |
| 1.0 % | 0.0996 | PASS | 0.0 % |
| 2.0 % | 0.0996 | PASS | 0.0 % |
| 3.0 % | 0.0996 | PASS | 0.0 % |
| 4.0 % | 0.0996 | PASS | 0.0 % |
| 5.0 % | 0.0996 | PASS | 0.0 % |
| 10.0 % | 0.0996 | PASS | 0.0 % |

TV drift is constant at 0.0996 across all caps; the 3% cap used in the main run is non-binding.

---

### Committed result artefacts

The repo ships the full main-run output under `mc_regime/outputs/runs/poc_v14_main/`:

- [`decision.json`](https://github.com/DavidVossebuerger/MC-Regime/blob/master/mc_regime/outputs/runs/poc_v14_main/decision.json) — GO/NO-GO, all L0–L4 aggregated losses, per-pair Sharpes
- [`sinkhorn_cap_sensitivity.csv`](https://github.com/DavidVossebuerger/MC-Regime/blob/master/mc_regime/outputs/runs/poc_v14_main/sinkhorn_cap_sensitivity.csv) — cap sweep
- [`test_results.json`](https://github.com/DavidVossebuerger/MC-Regime/blob/master/mc_regime/outputs/runs/poc_v14_main/test_results.json) — 24 Wilcoxon tests + BH-FDR
- [`transition_matrix_calibrated.npy`](https://github.com/DavidVossebuerger/MC-Regime/blob/master/mc_regime/outputs/runs/poc_v14_main/transition_matrix_calibrated.npy) — Sinkhorn output
- [`regime_windows.npy`](https://github.com/DavidVossebuerger/MC-Regime/blob/master/mc_regime/outputs/runs/poc_v14_main/regime_windows.npy) — 8 regime boundaries
- [`FINAL_REPORT.md`](https://github.com/DavidVossebuerger/MC-Regime/blob/master/mc_regime/outputs/runs/poc_v14_main/FINAL_REPORT.md) — v13→v14 audit trail

No pipeline re-run required to inspect numbers — the artefacts are committed to the repo.

---

### Reproduce

```bash
git clone https://github.com/DavidVossebuerger/MC-Regime.git
cd MC-Regime
pip install -e .

python3 -u mc_regime/scripts/run_poc.py \
    --replications 1999 \
    --pairs all \
    --pricer beer \
    --calibrate-target empirical \
    --output outputs/runs/main
```

23 unit tests, ~25 minutes wall-clock on a modern multi-core CPU.

<script>
document.addEventListener('DOMContentLoaded', function() {
  if (!window.Chart) return;
  fetch('/data/mc_regime_data.json').then(r => r.json()).then(data => {
    const css = getComputedStyle(document.body);
    const accent = (css.color || '#333').trim();
    const muted = accent.replace('rgb', 'rgba').replace(')', ', 0.55)');

    // L0 chart
    const lossCtx = document.getElementById('mc-loss-chart');
    if (lossCtx) {
      const l0 = data.L0_losses;
      const labels = Object.keys(l0);
      const values = labels.map(k => l0[k]);
      const isMarkov = k => k === 'Regime-Markov';
      new Chart(lossCtx, {
        type: 'bar',
        data: { labels: labels, datasets: [{
          data: values,
          backgroundColor: labels.map(l => isMarkov(l) ? accent : muted),
          borderWidth: 0,
          barThickness: 36
        }]},
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: { legend: { display: false }, tooltip: { callbacks: { label: ctx => ctx.label + ': ' + ctx.parsed.y.toFixed(3) } } },
          scales: {
            y: { title: { display: true, text: 'Cross-pool L0 loss (lower = better)' }, beginAtZero: true, grid: { color: 'rgba(127,127,127,0.12)' } },
            x: { grid: { display: false } }
          }
        }
      });
    }

    // Per-pair Sharpe chart
    const pairCtx = document.getElementById('mc-pair-chart');
    if (pairCtx) {
      const labels = ['EURUSD', 'GBPUSD', 'USDJPY'];
      const values = labels.map(l => data.per_pair_sharpe[l.toLowerCase()]);
      new Chart(pairCtx, {
        type: 'bar',
        data: { labels: labels, datasets: [{
          data: values,
          backgroundColor: values.map(v => v >= 0 ? accent : muted),
          borderWidth: 0,
          barThickness: 50
        }]},
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: { legend: { display: false }, tooltip: { callbacks: { label: ctx => ctx.label + ': ' + ctx.parsed.y.toFixed(3) } } },
          scales: {
            y: { title: { display: true, text: 'BEER in-sample Sharpe' }, grid: { color: 'rgba(127,127,127,0.12)' } },
            x: { grid: { display: false } }
          }
        }
      });
    }

    // (cap-sensitivity chart removed: TV drift is constant across all caps;
    //  a table communicates the result with no visual fabrication.)
  });
});
</script>
