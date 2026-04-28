# Reward Model Training and Evaluation

In this chapter, we will cover the training and evaluation of scalar reward models using multi-node containerized training on AWS. We will use the training infrastructure set up in the previous chapters to train Bradley-Terry reward models on a pairwise preference dataset using the Axolotl library. We will then evaluate the trained models using pairwise accuracy on the RewardBench dataset. By the end of this chapter, you will have a trained reward model that can be used for applications such as reinforcement learning with human feedback (RLHF) or as a component in larger systems that require reward modeling.

## Steps

**Step 1:** Prepare the local containerized training environment. This involves converting the Docker containers created in the first chapter to the enroot format. Enroot is a handy toolkit that turns traditional Docker containers into unprivileged sandboxes freed from the resource limits typically imposed on Docker containers by default. Its rootless model, along with plugins for popular cluster managers like Slurm, makes it a great fit for containerized multi-node training on the cloud.

For this step, you can either convert your own container or use the pre-built Docker environment at [ineil77/axolotl-rm:23032026](https://hub.docker.com/repository/docker/ineil77/axolotl-rm/tags/22032026/).

```bash
# Import the docker container to enroot format
enroot import docker://ineil77/axolotl-rm:23032026 --o axolotl-rm.sqsh
```

```bash
# Create the final enroot environment
enroot create --name Axolotl axolotl-rm.sqsh
```

These steps must be performed on all EC2 instances that will be used for training, so it is recommended to create a script to automate this process.

**Step 2:** Prepare the training scripts and axolotl configuration files.

```bash
# Create the training directory and add the configuration files
mkdir -p /mnt/fsx/Reward_Modeling/
touch /mnt/fsx/Reward_Modeling/32B_RM_Config.yaml
```

We have provided an example Axolotl configuration file for training a Bradley-Terry reward model on the pairwise preference dataset at [32B_RM_Config.yaml](./32B_RM_Config.yaml). We use the post-trained Qwen3-32B model as the base language model and train it using the Bradley-Terry loss with a magnitude regularization penalty. We use the GeneralPreference dataset, which contains a variety of validated preferences in the code, math, and general domains compiled from existing open-source data. You can also use your own datasets and customize the configuration file as needed for your specific setup.

**Step 3:** Launch the training job using the `enroot start` command. This starts the training process on the cluster using the enroot environment created in Step 1. You must run this command once on each EC2 instance used for training. Designate one instance as the master node and specify its private (not public) IP address as the rendezvous endpoint. All other nodes will connect to it to coordinate the training process. The private IP address can be found in the EC2 console under the "Networking" section of the instance details, or on the instance itself using the `hostname -I` command.

```bash
# Start the training job using enroot (Node 1 - Master Node)
enroot start --root \
    --rw \
    --mount /mnt/fsx/:/mnt/fsx/ \
    --env CUDA_VISIBLE_DEVICES="0,1,2,3,4,5,6,7" \
    --env HF_TOKEN=<YOUR_HF_TOKEN> \
    --env NCCL_NVLS_ENABLE=0 \
    --env NCCL_P2P_LEVEL=NVL \
    --env PYTORCH_CUDA_ALLOC_CONF="expandable_segments:True" \
    --env TORCH_DIST_INIT_BARRIER=1 \
    --env WANDB_API_KEY=<YOUR_WANDB_KEY> \
    Axolotl torchrun --nnodes 2 \
        --nproc_per_node 8 \
        --node_rank 0 \
        --rdzv_id "axolotl-rm" \
        --rdzv_endpoint="<MASTER_NODE_IP>:<MASTER_NODE_PORT>" \
        --rdzv_backend "c10d" -m axolotl.cli.train --config /mnt/fsx/Reward_Modeling/32B_RM_Config.yaml
```

```bash
# Start the training job using enroot (Node 2 - Worker Node)
enroot start --root \
    --rw \
    --mount /mnt/fsx/:/mnt/fsx/ \
    --env CUDA_VISIBLE_DEVICES="0,1,2,3,4,5,6,7" \
    --env HF_TOKEN=<YOUR_HF_TOKEN> \
    --env NCCL_NVLS_ENABLE=0 \
    --env NCCL_P2P_LEVEL=NVL \
    --env PYTORCH_CUDA_ALLOC_CONF="expandable_segments:True" \
    --env TORCH_DIST_INIT_BARRIER=1 \
    --env WANDB_API_KEY=<YOUR_WANDB_KEY> \
    Axolotl torchrun --nnodes 2 \
        --nproc_per_node 8 \
        --node_rank 1 \
        --rdzv_id "axolotl-rm" \
        --rdzv_endpoint="<MASTER_NODE_IP>:<MASTER_NODE_PORT>" \
        --rdzv_backend "c10d" -m axolotl.cli.train --config /mnt/fsx/Reward_Modeling/32B_RM_Config.yaml
```

**Step 4:** Once training completes, evaluate the trained reward model by launching an evaluation job with `enroot start`. This evaluates the model on the RewardBench dataset using pairwise accuracy as the metric.

```bash
# Start the evaluation job using enroot (Can be run on any node)
enroot start --root \
    --rw \
    --mount /mnt/fsx/:/mnt/fsx/ \
    --env CUDA_VISIBLE_DEVICES="0,1,2,3,4,5,6,7" \
    --env HF_TOKEN=<YOUR_HF_TOKEN> \
    --env NCCL_NVLS_ENABLE=0 \
    --env NCCL_P2P_LEVEL=NVL \
    --env PYTORCH_CUDA_ALLOC_CONF="expandable_segments:True" \
    --env TORCH_DIST_INIT_BARRIER=1 \
    --env WANDB_API_KEY=<YOUR_WANDB_KEY> \
    Axolotl rewardbench --model=<PATH_TO_TRAINED_MODEL_CHECKPOINT> \
        --batch-size=16
```

Our 32B reward model achieves a pairwise accuracy of nearly 92% on the RewardBench dataset, which is a strong result. This indicates that the model has effectively learned to capture human preferences, and it can be used for various downstream applications that require reward modeling. You can compare the performance of your trained model with other models or baselines to further assess its capabilities.

![Reward Model Evaluation Results](../../Assets/5_RM_Training%20Evaluation/5_RewardBench_Result.png)
