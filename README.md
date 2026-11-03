# Cyber Attack Detection with Graph Neural Networks

An end-to-end defensive security research project that turns network flows into a communication graph, trains a hybrid GraphSAGE/GAT model to classify hosts as benign or malicious, and exposes model attention in an interactive forensic explorer.

[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![PyTorch](https://img.shields.io/badge/PyTorch-2.x-EE4C2C?logo=pytorch&logoColor=white)](https://pytorch.org/)
[![PyTorch Geometric](https://img.shields.io/badge/PyTorch%20Geometric-GNN-3C2179)](https://pytorch-geometric.readthedocs.io/)
[![D3.js](https://img.shields.io/badge/Visualization-D3.js-F9A03C?logo=d3.js&logoColor=white)](https://d3js.org/)
[![Dataset](https://img.shields.io/badge/Dataset-CICIDS%202017-263238)](https://www.unb.ca/cic/datasets/ids-2017.html)

## Overview

Rather than treating each flow as an isolated record, the pipeline models IP addresses as nodes and source-to-destination communication as directed edges. A node-level classifier learns from both engineered traffic features and its graph neighborhood.

```text
CICIDS-style flow records
        -> cleaning and feature engineering
        -> directed host communication graph
        -> GraphSAGE -> GraphSAGE -> multi-head Graph Attention
        -> benign / malicious node scores
        -> attention subgraph and interactive investigation view
```

The repository includes the training and evaluation code, a trained checkpoint, a serialized graph, explainability outputs, and a browser-based demo. The included artifacts let reviewers explore the result without first downloading the raw dataset or training the model.

## What It Demonstrates

- **Graph construction:** Builds host-level features and directed communication edges from flow records.
- **Imbalance-aware learning:** Uses positive-class weighting and early stopping based on validation PR-AUC.
- **Hybrid GNN:** Applies two GraphSAGE layers followed by a multi-head Graph Attention layer.
- **Evaluation:** Reports precision, recall, F1, PR-AUC, a classification report, and stratified k-fold results.
- **Explainability:** Extracts a k-hop neighborhood, exports edge attention, and generates visual reports.
- **Investigation UI:** D3.js explorer for inspecting node labels, model scores, features, and attention-weighted links.

## Project Structure

| File | Description |
| --- | --- |
| `data_pipeline.py` | Loads flow CSVs, engineers node features, and writes a PyTorch Geometric graph. |
| `train_gnn.py` | Defines and trains the GraphSAGE/GAT node classifier. |
| `cross_validate_gnn.py` | Runs stratified k-fold model evaluation. |
| `phase5_evaluation_explainability.py` | Evaluates the saved model and exports explainability artifacts. |
| `index.html`, `app.js`, `styles.css` | Interactive D3.js forensic graph explorer. |
| `cyber_graph_data.pt`, `best_cyber_gnn.pth` | Example graph and trained model checkpoint. |
| `forensic_graph.json` | Graph data loaded by the browser demo. |
| `*_graph.png`, `explainability_report.png` | Generated evaluation and explainability figures. |

## Quick Start

### Requirements

- Python 3.10 or newer
- PyTorch and PyTorch Geometric (CPU or CUDA build)
- Raw CICIDS 2017 data only if rebuilding the graph from source

Create an environment and install the project dependencies:

```bash
python -m venv .venv
```

```powershell
# Windows PowerShell
.\.venv\Scripts\Activate.ps1
```

```bash
# macOS / Linux
source .venv/bin/activate
```

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

For GPU use, install the PyTorch build that matches your CUDA setup by following the [official PyTorch instructions](https://pytorch.org/get-started/locally/). See the [PyTorch Geometric installation guide](https://pytorch-geometric.readthedocs.io/en/latest/install/installation.html) if your platform needs a specific build.

### Explore the included demo

From the repository root, start a local web server:

```bash
python -m http.server 8000
```

Open [http://localhost:8000](http://localhost:8000). Select nodes to inspect their labels, predicted disposition, attack confidence, and engineered features. The included `forensic_graph.json` powers this view.

### Rebuild and train

Raw flow CSVs are not included. Download CICIDS 2017 from the [Canadian Institute for Cybersecurity](https://www.unb.ca/cic/datasets/ids-2017.html), place the required files in `data/`, and check `PipelineConfig.dataset_files` and `data_path` near the bottom of `data_pipeline.py` to match your local files.

Then run the pipeline and training workflow:

```bash
python data_pipeline.py
python train_gnn.py
python cross_validate_gnn.py --graph cyber_graph_data.pt --folds 5 --epochs 40
python phase5_evaluation_explainability.py
```

The pipeline writes `cyber_graph_data.pt`; training writes `best_cyber_gnn.pth`; the explainability script updates `forensic_graph.json` and the report figures. Dataset variants, preprocessing, split seeds, hardware, and decision threshold can all affect results.

## Evaluation Artifacts

The checked-in figures are generated outputs, not claims of production performance.

![Explainability report](explainability_report.png)

![Performance split](performance_split_graph.png)

![Literature comparison](literature_comparison_graph.png)

For reproducible reporting, run the evaluation scripts on the dataset version and configuration you intend to discuss, then cite the resulting metrics and split details rather than relying on screenshots alone.

## Limitations and Responsible Use

- This is an experimental, offline research/demo workflow, not a production intrusion detection system.
- Predictions are decision-support signals; validate alerts with independent evidence before taking action.
- Use only on data and networks you are authorized to analyze.
- Raw packet/flow captures are intentionally excluded from Git. The `data/` CSVs can exceed GitHub's file-size limits and should be obtained from their source under its applicable terms.
- No software license has been selected for this repository. Add one before reuse or redistribution if you intend to grant explicit permissions.

## Resume Summary

**Cyber Attack Detection with Graph Neural Networks** | Python, PyTorch Geometric, scikit-learn, D3.js

Built an end-to-end network-flow analytics pipeline that constructs directed host communication graphs, trains an imbalance-aware GraphSAGE/GAT node classifier, evaluates performance with PR-AUC and stratified cross-validation, and visualizes attention-based forensic neighborhoods in an interactive web application.