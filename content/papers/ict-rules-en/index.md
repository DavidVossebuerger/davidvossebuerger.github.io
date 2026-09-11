---
title: "Auditing ICT/PO3 FX Heuristics: An Independent Empirical Test of the Daily Swing Framework"
date: 2026-05-03
lastmod: 2026-06-12
author: ["David Vossebürger"]
description: "Independent empirical audit of PO3-inspired ICT trading heuristics (Daily Swing Framework, Composite) on six instruments over 18 years of M30 data. Negative-result study."
summary: "Independent audit of PO3-inspired ICT trading folklore on six instruments over 18 years of M30 data. Result: no risk-adjusted edge under realistic retail transaction costs."
editPost:
    URL: "https://papers.ssrn.com/sol3/papers.cfm?abstract_id=6700099"
    Text: "SSRN 6700099"

---

<a class="repo-card" href="https://github.com/DavidVossebuerger/po3-ict-backtesting" target="_blank" rel="noopener">
  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16" width="20" height="20" fill="currentColor"><path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0 0 16 8c0-4.42-3.58-8-8-8z"/></svg>
  <div class="repo-meta">
    <span class="repo-name">DavidVossebuerger/po3-ict-backtesting</span>
    <span class="repo-action">View source code on GitHub →</span>
  </div>
</a>

{{< ssrn id="6700099" >}}

---

<div class="audit-banner">

**Audit framing.** This is an independent empirical test of two strategies popularised inside the ICT/PO3 trading community — the Daily Swing Framework and a Composite signal aggregator. No commercial or community affiliation with ICT/PO3. Tests whether the framework survives realistic retail transaction costs.

</div>

---

### Abstract

This paper evaluates two rule-based ICT trading strategies — the Daily Swing Framework and a Composite signal aggregator — within a reproducible backtesting framework. The empirical basis is a multi-asset run across six instruments (EURUSD, GBPUSD, USDJPY, XAUUSD, USA500IDXUSD, USATECHIDXUSD) on M30 bid data from 2007 to 2025 (Dukascopy), with calibration, out-of-sample (OOS), and forward phases examined separately per instrument.

Neither strategy shows robust outperformance in its current parameterization. The daily-swing framework produced annualised Sharpe ratios between −3.89 and −0.35 across the six instruments, with maximum drawdowns of 100–227 %. Composite delivered Sharpe ratios of −1.24 to −0.53 with drawdowns of 47–81 %. A cost-attribution analysis reveals that under idealized (frictionless) execution the daily-swing framework achieves a Sharpe of +0.81 on EURUSD, but retail-level spread and slippage fully erase this edge.

Initial drafting was assisted by large language models; all quantitative analysis and interpretations were verified by the author.

---

### Headline result

> Across six instruments and 12 strategy-instrument combinations, the daily-swing framework and the composite signal aggregator deliver negative risk-adjusted returns on every single pairing. Under realistic retail costs, both strategies lose to a naive IID benchmark.

---

### Equity curves — EURUSD, 2007–2025

<figure class="chart-figure">
<p class="chart-title">Cumulative PnL (EURUSD, weekly bucket, 10 000 EUR starting capital)</p>
<div class="chart-canvas-wrap tall"><canvas id="ict-equity-chart"></canvas></div>
<p class="chart-caption">Click the legend to toggle strategies. Both strategies trend toward zero despite the buy &amp; hold baseline holding its initial capital. Per-trade data is only committed to the source repo for EURUSD; the other five instruments report summary metrics in <code>results/summary.csv</code> only.</p>
</figure>

<script>
document.addEventListener('DOMContentLoaded', function() {
  if (!window.Chart) return;
  const ctx = document.getElementById('ict-equity-chart');
  if (!ctx) return;
  fetch('/data/po3_equity_curves.json').then(r => r.json()).then(data => {
    const labels = data.weeks.map(w => w.date);
    const ds = data.weeks.map(w => w.daily_swing);
    const co = data.weeks.map(w => w.composite);
    const css = getComputedStyle(document.body);
    const accent = (css.color || '#333').trim();
    const muted = accent.replace('rgb', 'rgba').replace(')', ', 0.55)');
    new Chart(ctx, {
      type: 'line',
      data: { labels: labels, datasets: [
        { label: 'Daily Swing', data: ds, borderColor: accent, backgroundColor: accent, pointRadius: 0, borderWidth: 1.4, tension: 0.1 },
        { label: 'Composite', data: co, borderColor: muted, backgroundColor: muted, pointRadius: 0, borderWidth: 1.4, tension: 0.1 }
      ]},
      options: {
        responsive: true,
        maintainAspectRatio: false,
        interaction: { mode: 'index', intersect: false },
        plugins: { legend: { position: 'bottom', labels: { boxWidth: 14 } } },
        scales: {
          y: { type: 'logarithmic', title: { display: true, text: 'Equity (EUR, log scale)' }, grid: { color: 'rgba(127,127,127,0.12)' } },
          x: { ticks: { maxTicksLimit: 8, autoSkip: true }, grid: { display: false } }
        }
      }
    });
  });
});
</script>

---

### Multi-asset results (initial capital 10 000 EUR)

<figure class="chart-figure">
<p class="chart-title">Annualised Sharpe ratio by instrument and strategy</p>
<div class="chart-canvas-wrap tall"><canvas id="ict-sharpe-chart"></canvas></div>
<p class="chart-caption">All 12 strategy–instrument combinations produce negative Sharpe ratios. The X-axis lists the six instruments in the same order as the table below.</p>
</figure>

| Symbol | Strategy | Trades | Win Rate | Profit Factor | Max DD | Sharpe |
|---|---|---:|---:|---:|---:|---:|
| EURUSD | Daily Swing | 1,624 | 36.6 % | 0.30 | 100.0 % | −3.50 |
| EURUSD | Composite | 197 | 46.2 % | 0.31 | 69.8 % | −1.00 |
| GBPUSD | Daily Swing | 1,545 | 35.2 % | 0.25 | 100.0 % | −3.46 |
| GBPUSD | Composite | 218 | 44.5 % | 0.24 | 80.7 % | −1.24 |
| USDJPY | Daily Swing | 1,617 | 31.9 % | 0.21 | 100.0 % | −3.89 |
| USDJPY | Composite | 189 | 49.7 % | 0.30 | 71.0 % | −0.94 |
| XAUUSD | Daily Swing | 1,508 | 38.4 % | 0.47 | 99.8 % | −2.47 |
| XAUUSD | Composite | 185 | 55.1 % | 0.46 | 49.5 % | −0.72 |
| USA500 | Daily Swing | 151 | 39.7 % | 0.13 | 100.0 % | −1.60 |
| USA500 | Composite | 108 | 53.7 % | 0.18 | 66.1 % | −0.53 |
| USATECH | Daily Swing | 92 | 35.9 % | 0.08 | 100.0 % | −1.46 |
| USATECH | Composite | 114 | 53.5 % | 0.38 | 46.7 % | −0.67 |

Source: [`results/multi_asset_validation.json`](https://github.com/DavidVossebuerger/po3-ict-backtesting/blob/main/results/multi_asset_validation.json) in the source repository.

<script>
document.addEventListener('DOMContentLoaded', function() {
  if (!window.Chart) return;
  const labels = ['EURUSD','GBPUSD','USDJPY','XAUUSD','USA500','USATECH'];
  const ds = [-3.50,-3.46,-3.89,-2.47,-1.60,-1.46];
  const comp = [-1.00,-1.24,-0.94,-0.72,-0.53,-0.67];
  const css = getComputedStyle(document.body);
  const accent = (css.color || '#333').trim();
  const muted = accent.replace('rgb', 'rgba').replace(')', ', 0.55)');
  new Chart(document.getElementById('ict-sharpe-chart'), {
    type: 'bar',
    data: { labels: labels, datasets: [
      { label: 'Daily Swing', data: ds[0], backgroundColor: accent },
      { label: 'Composite',   data: ds[1], backgroundColor: muted }
    ]},
    options: {
      responsive: true,
      maintainAspectRatio: false,
      plugins: { legend: { position: 'bottom', labels: { boxWidth: 14 } } },
      scales: {
        y: { title: { display: true, text: 'Annualised Sharpe' }, grid: { color: 'rgba(127,127,127,0.12)' } },
        x: { grid: { display: false } }
      }
    }
  });
});
</script>

---

### Cost-attribution finding

<figure class="chart-figure">
<p class="chart-title">EURUSD Daily Swing Framework — Sharpe ratio under execution cost scenarios</p>
<div class="chart-canvas-wrap short"><canvas id="ict-cost-chart"></canvas></div>
<p class="chart-caption">Frictionless execution would have produced a positive Sharpe of +0.81. Retail-level spread and slippage (2 bps + 1 bps) flip it to −3.50. Source: cost-attribution scenario in <code>results/multi_asset_validation.json</code>.</p>
</figure>

Under idealized frictionless execution, the Daily Swing Framework on EURUSD achieves a Sharpe of **+0.81**. Under realistic retail conditions (spread 2 bps, slippage 1 bps), the same strategy delivers a Sharpe of **−3.50**. Execution costs erase the rule edge on EURUSD — not because the rules are wrong, but because the strategy does not survive its own turnover.

This pattern repeats across all six instruments and both strategies. It is the main finding.

<script>
document.addEventListener('DOMContentLoaded', function() {
  if (!window.Chart) return;
  const ctx = document.getElementById('ict-cost-chart');
  if (!ctx) return;
  const css = getComputedStyle(document.body);
  const accent = (css.color || '#333').trim();
  const data = {
    labels: ['Frictionless', 'Retail (2+1 bps)', 'Conservative (2.5+2.5 bps)'],
    datasets: [{
      data: [0.81, -3.50, -3.65],
      backgroundColor: [accent, accent, accent],
      borderWidth: 0,
      barThickness: 38
    }]
  };
  new Chart(ctx, {
    type: 'bar',
    data: data,
    options: {
      indexAxis: 'y',
      responsive: true,
      maintainAspectRatio: false,
      plugins: { legend: { display: false }, tooltip: { callbacks: { label: ctx => 'Sharpe: ' + ctx.parsed.x.toFixed(2) } } },
      scales: {
        x: { title: { display: true, text: 'Annualised Sharpe' }, grid: { color: 'rgba(127,127,127,0.12)' } },
        y: { grid: { display: false } }
      }
    }
  });
});
</script>

---

### Reproduce

```bash
git clone https://github.com/DavidVossebuerger/po3-ict-backtesting.git
cd po3-ict-backtesting
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# Single-asset quick run (~minutes)
python -m backtesting_system.main --symbols EURUSD --quick

# Full multi-asset run (~3h on 4-core VPS)
python -m backtesting_system.main \
    --symbols EURUSD,GBPUSD,USDJPY,XAUUSD,USA500IDXUSD,USATECHIDXUSD
```

The repo includes 23 unit tests, walk-forward analysis across 7 windows, parameter sensitivity over 18 configs, Monte Carlo resampling (1 000 iterations), and stress tests.
