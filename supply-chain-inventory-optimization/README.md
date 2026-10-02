# Supply Chain Inventory Optimization

An independent data science project that explores distribution network inventory and service-level optimization using synthetic FMCG data.

## Project contents

- `notebook_or_src/supply_chain_inventory_optimization.ipynb`: analysis, simulation, charts, and recommendation logic with saved results.
- `outputs/`: DC–SKU recommendations, holdout comparisons, validation trade-off data, and the summary chart.
- `data/supply_chain_dataset.csv`: synthetic supply chain data covering 6 distribution centers, 15 SKUs, and 104 weeks.
- `supply-chain-inventory-optimization.pdf`: project presentation and business findings.

## Main finding

The recommended 95th-percentile protection-demand policy reached **97.9% modeled unit fill** on the final 26 weeks, with **549 average on-hand units per DC–SKU**. The comparable modeled fixed-stock policy reached **99.4% fill** with **748 units**. This trades 1.6 percentage points of service for about 27% less physical inventory while remaining above the 95% fill target. The recorded current operation had 96.2% fill; its exact ordering logic is unavailable, so the modeled comparison should be validated through a live pilot.

## Method and assumptions

1. Use weeks 1–52 for each DC–SKU's demand and lead-time distributions; weeks 53–78 for sensitivity analysis; and weeks 79–104 for the chronological backtest.
2. Model weekly review. An order arrives after `ceil(supplier_lead_time_days / 7)` weeks. Protection demand covers that lead time plus one review week.
3. Bootstrap demand and lead-time observations independently to estimate a 95th-percentile protection-demand point. Recommended safety stock equals that point less expected protection demand, floored at zero.
4. Reconstruct the current point as expected protection demand plus the recorded fixed safety stock. Simulate current and recommended points with identical weekly ordering mechanics, actual demand and realized lead times, a 13-week warm-up, and lost sales when stock runs out.
5. Report unit fill rate, stockout-week rate, and mean physical on-hand inventory. Monetary working capital cannot be computed without unit costs.

An initial leaner 80th-percentile candidate met 95% fill on validation but missed it on holdout. The 95th-percentile recommendation was adopted after inspecting that result, so its holdout estimate is **exploratory**, not an untouched final test. The actual current operation also differs from the reconstructed current policy. Confirm service, stock, ordering rules, and costs in a pilot before rollout.

## How to run

From this directory, open `notebook_or_src/supply_chain_inventory_optimization.ipynb` and run all cells using Python 3.12. Required packages are `numpy`, `pandas`, `matplotlib`, `reportlab`, and `IPython`. The notebook reads the supplied CSV and recreates the project outputs. It can also be run with:

```bash
jupyter nbconvert --to notebook --execute --inplace notebook_or_src/supply_chain_inventory_optimization.ipynb
```

## AI-tool disclosure

ChatGPT/Codex assisted with notebook code, the simulation design, chart and memo generation, and README drafting. The assumptions and trade-offs are stated in the notebook so they can be reviewed and defended.
