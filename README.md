# LineVD Reproduction: Statement-Level Vulnerability Detection

Specialized project (Đồ án chuyên ngành), Information Security program, University of Information Technology (UIT), VNU-HCM.

This project reproduces and experiments with **LineVD**, a graph neural network that detects vulnerable C/C++ functions and pinpoints the vulnerable **statements** inside them. It trains and evaluates the model on the **Big-Vul** dataset of real-world vulnerabilities.

> **Based on [LineVD](https://github.com/davidhin/linevd)** by David Hin et al. (MIT License).
> Paper: *LineVD: Statement-level Vulnerability Detection using Graph Neural Networks*, MSR 2022 ([arXiv:2203.05181](https://arxiv.org/abs/2203.05181)).
> Most of the code in `sastvd/` comes from the original repository. My changes are listed below.

## What LineVD does

1. **Code graphs**: Joern turns each function into a Program Dependence Graph (PDG), where each node is a statement.
2. **Statement features**: each statement is embedded with CodeBERT (GloVe and Doc2Vec are also supported).
3. **Graph learning**: a Graph Attention Network (GAT) propagates information along control and data dependencies.
4. **Multi-task output**: the model predicts both whether the function is vulnerable and which statements are vulnerable.

## My contributions

- **Ran LineVD without Singularity.** Removed the `singularity exec` wrapper (`sastvd/__init__.py`) so the pipeline runs directly on a Linux machine or in Docker.
- **Ported the code to newer library versions:**
  - PyTorch Lightning: `ModelCheckpoint` import and `accelerator="auto"` instead of `gpus=1`.
  - DGL: `DataLoader` with `MultiLayerFullNeighborSampler` instead of the removed `NodeDataLoader`.
  - Ray Tune: removed `TuneReportCallback` from training loops.
- **Fitted experiments to limited hardware.** Reduced the Ray Tune search size and allowed CPU-only trials.
- **Added standalone training scripts** (no Ray Tune) to compare model settings:

| Script | Setting |
|---|---|
| `sastvd/scripts/train_demo.py` | Statement level, CodeBERT, 2-layer GAT, line + method multi-task |
| `sastvd/scripts/train_demo_2.py` | Same setting with GCN instead of GAT |
| `sastvd/scripts/train_demo_method.py` | GloVe embeddings, MLP-only baseline |
| `sastvd/scripts/getdata.py` | Method level, GloVe, 2-layer GAT |
| `sastvd/scripts/demo_ray.py` | Ray Tune smoke test |
| `sastvd/scripts/train_best.ipynb` | Notebook run of the tuned configuration |

## Repository structure

```text
├── sastvd/
│   ├── codebert/      # CodeBERT feature extractor
│   ├── helpers/       # Joern, datasets, embeddings, metrics
│   ├── ivdetect/      # IVDetect baseline
│   ├── linevd/        # LineVD model, data module, training
│   └── scripts/       # data preparation, training and research-question scripts
├── storage/external/  # dataset splits and Joern script
├── tests/             # environment and Joern tests
├── cli.sh             # helper CLI (initialise / run)
└── get_data.sh        # downloads Big-Vul and Joern v1.1.260
```

## Getting started

Requirements: Linux, Python 3.10, Java (for Joern), and a CUDA GPU (recommended).

```bash
pip install -r requirements.txt
bash cli.sh -p initialise
```

The initialise step installs the `sastvd` package and downloads the Big-Vul dataset and Joern.

Prepare the data and build the graphs:

```bash
python sastvd/scripts/prepare.py
python sastvd/scripts/getgraphs.py
```

Train a model, for example the CodeBERT + GAT setting:

```bash
python sastvd/scripts/train_demo.py
```

Results are written to `storage/outputs/`.

## Credits and license

- Original code: [davidhin/linevd](https://github.com/davidhin/linevd), Copyright (c) 2021 David Hin, released under the MIT License (see [LICENSE](LICENSE)).
- Dataset: Big-Vul, Fan et al., *A C/C++ Code Vulnerability Dataset with Code Changes and CVE Summaries*, MSR 2020.
