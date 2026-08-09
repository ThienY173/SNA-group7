# When Does Sparse Relationalization of Tabular Metadata Help Popularity Prediction?

Code and results for the FDSE 2026 paper *"When Does Sparse Relationalization
of Tabular Metadata Help Popularity Prediction? A Controlled Study of Graph
Utility, Attribution, and Operating-Point Shifts Across Six Platforms"*
(Thien-Y Nguyen-Thai, Ai-Nu Huynh-Tran, Hung-Nghiep Tran — University of
Information Technology, VNU-HCM).

When a popularity-prediction dataset is a flat table, graph structure must
be manufactured rather than observed, and it is unclear when such sparse
metadata-derived graphs add predictive value beyond a tuned tabular model.
This repository evaluates four graph constructions — spanning relation
type, sparsification rule and edge weighting — against a metadata-only
XGBoost baseline, across six platform-labeled subsets of a curated
synthetic corpus (Facebook, Instagram, Reddit, TikTok, Twitter, YouTube),
over five seeds with paired per-seed deltas. The findings: utility is
small and conditional, apparent gains are largely operating-point shifts
rather than ranking improvements, and graph features can absorb
substantial model attribution without providing commensurate independent
predictive value.

## Data

`multi_platform_social_sentiment_evolution.csv` is the raw public dataset
(Kaggle, Gokhale, *Multi-Platform Social Sentiment Evolution*):
150,000 posts across six platforms, 31 raw attributes, no missing values.
Source: https://www.kaggle.com/datasets/sohumgokhale/multi-platform-social-sentiment-evolution

## Repository structure

```
.
├── data_preprocess/     # Feature engineering, leakage-column removal,
│                        # label construction (top-20% engagement per
│                        # platform), one-hot encoding, per-seed stratified
│                        # 80/20 splits (data_preprocess.ipynb, plus
│                        # processed_data/graph_data and processed_data/ml_data)
├── Network_graph/       # Per-platform notebooks (e.g. Facebook/facebook_graph.ipynb)
│                        # sharing graph_lib.py, which implements:
│                        #   - homophily edge construction (chain rule,
│                        #     Algorithm 1), shared by unweighted_homophily
│                        #     and data_driven_homophily
│                        #   - mutual-information relation weighting +
│                        #     permutation-null bias correction
│                        #     (training-partition only)
│                        #   - similarity k-NN graph construction
│                        #   - centrality features (degree, betweenness,
│                        #     closeness, PageRank, eigenvector, modularity
│                        #     class)
│                        #   - Node2Vec embeddings (D=64)
│                        #   - leakage audit (train/train, train/test,
│                        #     test/test edge counts)
│                        #   - threshold-tuned baseline (no_graph_tau)
│                        #   - paired per-seed deltas, classification,
│                        #     grouped SHAP, targeted ablation,
│                        #     error-transition analysis
│                        # results/ holds the per-platform result CSVs and plots
└── multi_platform_social_sentiment_evolution.csv
```

## Results (`Network_graph/results/`)

Per platform, the pipeline writes:

| Category | Contents |
|---|---|
| Metrics | Raw and summary accuracy/precision/recall/F1/ROC-AUC/PR-AUC per variant and seed, the threshold-tuned control (`no_graph_tau`), and paired deltas vs. `no_graph` with a positive/negative/trade-off classification |
| Graph diagnostics | Node/edge/degree statistics with the train–test leakage audit, and relation weights with raw vs. permutation-corrected mutual information |
| Interpretability | Targeted ablation, error-transition counts, and grouped SHAP attribution (summary + per-seed CSVs and plots) for the full `data_driven_homophily+node2vec64` variant |

## Reproducing the paper's tables

- **Table 1** (dataset & graph statistics) ← `Network_graph/results/{platform}_graph_statistics.csv`, averaged over the 5 seeds.
- **Table 3** (paired deltas, classification, threshold-tuned control) ← `{platform}_paired_deltas.csv` and `{platform}_no_graph_tau.csv`.
- **Table 4** (raw vs. permutation-corrected MI) ← `{platform}_mi_corrected.csv`.
- **Table 5** (grouped SHAP, ablation, error transitions) ← `{platform}_grouped_shap_summary.csv`, `{platform}_ablation_results.csv`, `{platform}_error_transitions.csv`, all for the `data_driven_homophily+node2vec64` variant.

## Fixed configuration (identical across platforms, seeds, and variants)

- **Seeds:** `{42, 123, 2024, 7, 99}`
- **Classifier:** XGBoost, `n_estimators=500`, `learning_rate=0.02`, `max_depth=6`, `min_child_weight=1`, `gamma=0.1`, `subsample=0.8`, `colsample_bytree=0.5`, `scale_pos_weight = n_neg/n_pos` (from the current training partition), `objective="binary:logistic"`
- **Label:** top-20% of total engagement, computed once per platform before splitting
- **Split:** stratified 80/20 per platform per seed, reused identically by all 5 variants
- **Homophily relation weights:** min–max normalized to $[0.5, 2.0]$ from training-partition mutual information (see the paper's Section 5.2 for the cardinality-bias finding)
- **Node2Vec:** $D=64$, walk length 20, 20 walks/node, window 5, min count 1, $p=q=1$

## Citation

If you use this code or these results, please cite:

```bibtex
@inproceedings{nguyenthai2026sparse,
  title     = {When Does Sparse Relationalization of Tabular Metadata Help
               Popularity Prediction? A Controlled Study of Graph Utility,
               Attribution, and Operating-Point Shifts Across Six Platforms},
  author    = {Nguyen-Thai, Thien-Y and Huynh-Tran, Ai-Nu and Tran, Hung-Nghiep},
  booktitle = {Proceedings of FDSE 2026},
  year      = {2026}
}
```

## License

Add a license (e.g., MIT for code, CC-BY for results) before or shortly
after submission — none is currently specified in this repository.
