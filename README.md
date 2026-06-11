# BRIDGE-Connectome

**BRIDGE: Brain Representation via Individualized Deep Generative Embedding for Normative Modeling of Connectomes**

This repository provides instructions for running **BRIDGE-Connectome** training using the supplied **Apptainer/Singularity container**.

The container includes the BRIDGE-Connectome source code and all required software dependencies. Experiment-specific resources such as configuration files, datasets, subject lists, checkpoints, and output directories are expected to remain outside the container and are mounted at runtime using bind mounts.

---

## Overview

A typical BRIDGE-Connectome experiment requires:

- A YAML configuration file
- A connectome dataset (`.pkl`)
- A geodesic distance dataset (`.pkl`)
- Training subject list (`.txt`)
- Testing subject list (`.txt`)

The general workflow is:

1. Prepare a YAML configuration file.
2. Organize datasets and subject lists.
3. Create output and cache directories.
4. Launch the BRIDGE container.
5. Monitor training logs and checkpoints.

---

## Repository Layout

A recommended project structure is shown below:

```text
project/
├── configs/
│   └── my_bridge_experiment.yaml
│
├── data/
│   ├── CAR_Schaefer2018_200.pkl
│   ├── geodist_Schaefer200.pkl
│   └── car_lists/
│       ├── train_list_fold0.txt
│       └── test_list_fold0.txt
│
├── checkpoints/
├── pyg_cache/
└── temp_result/
```

---

## Required Input Files

### 1. YAML Configuration

The YAML file defines:

- Model architecture
- Training parameters
- Optimizer settings
- Learning-rate schedule
- Dataset locations
- Output directories

Example:

```yaml
manual_seed: 1
device: cpu

model:
  name: BRIDGE
  in_channels: 200
  out_channels: 6
  feature_channels: 40
  num_nodes: 200
  restore: null

trainer:
  checkpoint_dir: /output/checkpoints/my_bridge_experiment
  epochs: 500
  eval_score_higher_is_better: False

optimizer:
  learning_rate: 0.001
  weight_decay: 0.0001

lr_scheduler:
  name: MultiStepLR
  milestones: [50, 200]
  gamma: 0.1

loaders:
  name: CARSet
  loader_name: DataLoader
  root: /output/pyg_cache/my_bridge_experiment

  train_list: /data/car_lists/train_list_fold0.txt
  output_train: train_fold0.pkl
  train_val_ratio: [3, 1]

  test_list: /data/car_lists/test_list_fold0.txt
  output_test: test_fold0.pkl

  path_data:
    - /data/CAR_Schaefer2018_200.pkl
    - /data/geodist_Schaefer200.pkl

  target_name: [Age, Sex]
  feature_mask: [200, 30]
  batch_size: 36
```

---

### 2. Connectome Dataset

Example:

```text
/data/CAR_Schaefer2018_200.pkl
```

This file contains the precomputed connectome representations and metadata used during training.

---

### 3. Geodesic Distance Dataset

Example:

```text
/data/geodist_Schaefer200.pkl
```

This file contains graph geodesic distance information used during graph construction.

---

### 4. Subject Lists

Training and testing subject lists should contain one subject identifier per line:

```text
subject001
subject002
subject003
```

Example files:

```text
/data/car_lists/train_list_fold0.txt
/data/car_lists/test_list_fold0.txt
```

---

## Running BRIDGE-Connectome

### Create Output Directories

```bash
mkdir -p checkpoints/my_bridge_experiment/logs
mkdir -p pyg_cache/my_bridge_experiment
mkdir -p temp_result
```

### Local Execution

```bash
singularity run --no-home \
  --bind $(pwd)/configs:/configs \
  --bind $(pwd)/data:/data \
  --bind $(pwd):/output \
  bridge.sif \
  /configs/my_bridge_experiment.yaml
```

### Bind Mounts

| Host Directory | Container Path | Purpose |
|---------------|---------------|----------|
| `configs/` | `/configs` | YAML configuration files |
| `data/` | `/data` | Datasets and subject lists |
| Project root | `/output` | Checkpoints, logs, cache, outputs |

---

## Running on SLURM

Example job submission:

```bash
sbatch \
  --job-name=bridge_example \
  --cpus-per-task=8 \
  --mem=98G \
  --time=1-00:00:00 \
  --partition=all \
  --output="/path/to/project/temp_result/slurm_bridge_example_%j.out" \
  --wrap="singularity run --no-home \
    --bind /path/to/project/configs:/configs \
    --bind /path/to/project/data:/data \
    --bind /path/to/project:/output \
    /path/to/bridge.sif \
    /configs/my_bridge_experiment.yaml"
```

---

## Outputs

### Training Log

Training console output:

```text
/output/temp_result/my_bridge_experiment.txt
```

### Model Checkpoints

Saved model checkpoints:

```text
/output/checkpoints/my_bridge_experiment/
```

### TensorBoard Logs

TensorBoard event files:

```text
/output/checkpoints/my_bridge_experiment/logs/
```

Launch TensorBoard:

```bash
tensorboard --logdir checkpoints/my_bridge_experiment/logs
```

### PyTorch-Geometric Cache

Processed graph cache files:

```text
/output/pyg_cache/my_bridge_experiment/
```

These files are reused across runs to reduce preprocessing time.

---

## Running Multiple Experiments

When running multiple experiments simultaneously, use unique output locations for each experiment.

Example:

```yaml
trainer:
  checkpoint_dir: /output/checkpoints/fold1_experiment

loaders:
  root: /output/pyg_cache/fold1_experiment
```

Using separate directories prevents cache collisions and checkpoint overwrites.

---

## Best Practices

- Keep datasets outside the container image.
- Version-control YAML files alongside experiments.
- Use unique checkpoint directories for each run.
- Use unique PyTorch-Geometric cache directories for each run.
- Store SLURM logs in `temp_result/`.
- Regularly back up trained checkpoints.

---

## Container Contents

The BRIDGE container includes:

- BRIDGE-Connectome source code
- Python dependencies
- PyTorch
- PyTorch-Geometric dependencies
- Training and evaluation scripts

The following are **not** included and must be supplied by the user:

- YAML configuration files
- Connectome datasets
- Geodesic distance datasets
- Subject lists
- Output directories

---

## Citation

If you use BRIDGE-Connectome in your research, please cite the corresponding BRIDGE publication and any associated datasets used in your experiments.
