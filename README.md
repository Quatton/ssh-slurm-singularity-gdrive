# SSH + Slurm + Singularity + GDrive Example

This example demonstrates a full HPC workflow with:
- **SSH** to connect to a remote cluster
- **Slurm** for job scheduling
- **Singularity** for containerized execution
- **GDrive** for data and artifact storage

## Execution Chain

```
┌─────────────────────────────────────────────────────────┐
│ Your machine                                             │
│  └─► SSH ─────────────────────────────────────────────┐ │
│                                                        │ │
│ ┌──────────────────────────────────────────────────────┼─┤
│ │ HPC Cluster                                          │ │
│ │  └─► Slurm (sbatch) ──────────────────────────────┐ │ │
│ │                                                    │ │ │
│ │ ┌──────────────────────────────────────────────────┼─┼─┤
│ │ │ Compute Node                                     │ │ │
│ │ │  └─► Singularity ─────────────────────────────┐ │ │ │
│ │ │                                                │ │ │ │
│ │ │    ┌────────────────────────────────────────┐ │ │ │ │
│ │ │    │ Container                              │ │ │ │ │
│ │ │    │  /workspace/code  ← GitHub             │ │ │ │ │
│ │ │    │  /workspace/data  ← GDrive (readonly)  │ │ │ │ │
│ │ │    │  /workspace/runs  → GDrive (sync)      │ │ │ │ │
│ │ │    │                                        │ │ │ │ │
│ │ │    │  $ python train.py                     │ │ │ │ │
│ │ │    └────────────────────────────────────────┘ │ │ │ │
│ │ └──────────────────────────────────────────────────┘ │ │
│ └──────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

## Usage

```bash
# Submit training job to HPC
qwex run train

# Use a different runner
qwex run train --runner=local-container

# Override workflow args
qwex run train -- --epochs=200

# Check status
qwex status <run-id>

# Stream logs
qwex logs <run-id> --follow

# Cancel
qwex cancel <run-id>
```

## Setup

1. **SSH access**: Ensure you can `ssh hpc.university.edu`
2. **GDrive auth**: Place service account key at `~/.config/gdrive-sa.json`
3. **GitHub SSH**: Ensure your SSH key can clone the repo

## How it works

1. **Submit**: qwex connects via SSH and runs `sbatch` with a generated script
2. **Init**: Slurm job starts, clones code, downloads data via rclone
3. **Run**: Singularity executes your command in the container
4. **Sync**: On exit, uploads `/workspace/runs` to GDrive
5. **Query**: qwex can check status via GDrive (direct) or SSH+squeue (fallback)
