# Training Environment

This directory contains the Dockerfile and vendored source trees used to build the containerized training and evaluation environment for the tutorial. The resulting Docker image bundles everything needed for multi-node reward model training with EFA on AWS.

## Contents

| Path | Description |
|---|---|
| `Axolotl.Dockerfile` | Main Dockerfile that builds the training image (see below) |
| `axolotl/` | Modified fork of [Axolotl](https://github.com/axolotl-ai-cloud/axolotl) (v0.8.1) with Qwen3 chat model support |
| `xformers/` | Vendored [xFormers](https://github.com/facebookresearch/xformers) library for memory-efficient attention |
| `reward-bench/` | Vendored [RewardBench](https://github.com/allenai/reward-bench) (v0.1.5) evaluation suite |

## Docker Image

The `Axolotl.Dockerfile` builds on top of `nvcr.io/nvidia/pytorch:25.06-py3` and layers the following components:

1. **NVIDIA GDRCopy** (v2.5.1) -- GPU-direct RDMA memory copies
2. **AWS EFA Installer** (v1.45.1) -- Elastic Fabric Adapter drivers and libfabric
3. **AWS-OFI-NCCL** (v1.18.0) -- NCCL plugin for EFA/libfabric transport
4. **OpenMPI** -- configured for EFA with InfiniBand verbs removed
5. **DeepSpeed** (v0.18.8) -- distributed training engine
6. **Transformers** (v4.57.6) -- Hugging Face model library
7. **xFormers** -- built from the vendored source in `xformers/`
8. **Axolotl** -- built from the vendored source in `axolotl/`, installed with `ring-flash-attn` and `deepspeed` extras
9. **RewardBench** -- built from the vendored source in `reward-bench/`, installed with `v1` extras

### Building

From this directory:

```bash
docker build -t <your-tag> -f Axolotl.Dockerfile .
```

The three vendored source trees (`axolotl/`, `xformers/`, `reward-bench/`) are copied into the image at `/workspace/` and installed in editable mode. The build expects all three directories to be present alongside the Dockerfile.

### Supported GPU Architectures

The image is compiled with `TORCH_CUDA_ARCH_LIST="8.0 8.6 8.9 9.0 10.0 10.3+PTX"`, covering Ampere (A100), Ada Lovelace (L40S), Hopper (H100), and Blackwell GPUs.

## Usage with Enroot

The tutorial uses [enroot](https://github.com/NVIDIA/enroot) to convert this Docker image into an unprivileged sandbox for multi-node training. See [Chapter 5](../Chapters/5_RM_TrainingEvaluation/README.md) for details on importing and running the container with `enroot`.

A pre-built image is also available on Docker Hub at `ineil77/axolotl-rm:23032026`.
