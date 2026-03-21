# Multi-node reward model training on AWS with EFA and FSX storage

This tutorial details the process of setting up a multi-node reward model training environment on AWS using EFA and FSX storage. The tutorial is structured into several chapters, each covering a specific aspect of the setup process. By following these chapters, you will be able to create a robust and efficient training environment for your deep learning models. As a practical walkthrough, we will be outlining how to fine-tune an already pos-trained language model on the pointwise reward modeling task using the Bradley-Terry loss onjective. However, the setup process can be applied to a wide range of training tasks and models.

Ths practical tutorial will be using a modified version of the [Axolotl Framework v0.8.1](https://github.com/axolotl-ai-cloud/axolotl/tree/v0.8.1), with changes to support Qwen3 chat models. At the end of the tutorial, our infrastructure setup will look as follows:

![Multi-node training setup](./Assets/Setup_Outline.png)

## Contents

1. [Chapter 0](./Chapters/0_Setup_TestEC2/README.md): Setup test EC2 and create final EC2 AMI (Optional) and compile docker images
2. [Chapter 1](./Chapters/1_Setup_SG_EFA/README.md): Create security groups and cluster placement groups
3. [Chapter 2](./Chapters/2_Setup_FSX/README.md): Create shared FSX storage
4. [Chapter 3](./Chapters/3_Create_LaunchTemplate/README.md): Create launch template and maximize network bandwith and optimize Swap and mounting of FSX storage
5. [Chapter 4](./Chapters/4_Launch_Instance/README.md): Launch EC2 instances and bind public IPs and test EFA and FSX storage
6. [Chapter 5](./Chapters/5_RM_TrainingEvaluation/README.md): Initiate distributed training of the reward model and evaluate it on rewardbench

## Prerequisites

- An AWS account with appropriate permissions to create and manage EC2 instances, security groups, EFA, and FSX storage. The account must also have appropriately set service quota limits for the resources being used (e.g., EC2 instances, EFA interfaces, FSX storage).
- Basic knowledge of AWS services, particularly EC2, security groups, and EFA.
- Familiarity with deep learning frameworks and distributed training concepts is beneficial but not required.

## Where to go from here

This tutorial demonstrates how to set up a multi-node training environment on AWS using EFA and FSX storage using a practical example of fine-tuning a language model on a reward modeling task. However, the setup mainly leverages the GUI of the AWS console. Froa replicable setting of conducting large repeatable experiments, it is recommended to use infrastructure-as-code tools such as Terraform or AWS CloudFormation to automate the setup process. Additionally, it may be beneficial to setup a slurm cluster on top of the EC2 instances for easier management of distributed training jobs.
