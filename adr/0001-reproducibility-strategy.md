# ADR 0001: Reproducibility strategy for NorthStar models

## Status
Proposed

## Context
Currently, NorthStar Logistics lacks a formal system for model versioning and reproducibility. Models are stored in S3 with informal naming conventions (e.g., `eta_v2_FINAL.onnx`), making it impossible to audit which code version or dataset produced a specific artifact. There is no automated link between the training environment, the data snapshot, and the resulting model weights.

## Decision
We will implement a four-layer reproducibility strategy using industry-standard tooling:

1.  **Environment Layer**: All training and inference will occur within **Docker** containers. We will use **Poetry** for dependency management (`poetry.lock` pinned) and reference base images by their **SHA256 digest** rather than tags (e.g., `python:3.10@sha256:...`) to prevent silent upstream updates.
2.  **Data Layer**: We will adopt **DVC (Data Version Control)** to version datasets stored in S3. The `.dvc` pointer files, containing the MD5 hash of the data, must be committed to Git alongside the training code.
3.  **Code Layer**: **MLflow** will be used to track every training run. Each run will be automatically tagged with the **Git SHA** of the code used. We will enforce a "no-uncommitted-changes" policy for production training runs via CI/CD (GitHub Actions).
4.  **Randomness Layer**: All training scripts must accept a `random_seed` parameter from a centralized YAML configuration file. We will use `numpy.random.seed` and framework-specific deterministic flags (e.g., `torch.use_deterministic_algorithms(True)`) to ensure bit-level reproducibility where possible.

## Alternatives rejected
*   **S3 Timestamps for Data Versioning**: Rejected because it doesn't handle data mutations or deletions and lacks the atomic "code-data" link provided by DVC.
*   **Virtualenv/Requirements.txt**: Rejected because it doesn't capture system-level dependencies (C++ libraries, CUDA versions) which are critical for model consistency.
*   **Informal "Snapshot" Documentation**: Rejected because manual documentation is prone to human error and cannot be verified by automated audit tools.

## Consequences
*   **Operational Overhead**: Engineers must learn DVC and Poetry workflows. Every training run requires a Git commit of the DVC pointer.
*   **Storage Costs**: S3 costs will increase as we retain immutable snapshots of historical datasets.
*   **Build Times**: Enforcing strict Docker image creation and dependency locking will increase CI/CD pipeline duration.

## Revisit if
*   The team grows beyond 20 engineers, necessitating a move from a file-based DVC approach to a more complex Feature Store (e.g., Tecton or Feast).
*   Data volume exceeds 100TB, making the current DVC-over-S3 model cost-prohibitive for full snapshots.
