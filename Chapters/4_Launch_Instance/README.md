# Launch multiple EC2 instances for multi-node training and assign public IPs

In this chapter, we will launch multiple EC2 instances using the optimized launch template created in the previous chapter. We will also assign public IP addresses to these instances to enable SSH access and communication with other AWS services. This prepares us for the multi-node training process in the next chapter, where we will test the EFA and FSx storage setup and initiate distributed training.

## Steps

**Step 1:** Review all the settings for the launch template and confirm they are correct. Once satisfied, click "Create launch template". This template can now be used to launch EC2 instances optimized for multi-node training, with the correct network configuration, storage setup, and automated initialization. You can launch multiple instances at a time, and they will all be configured correctly. We will launch two instances for our reward model training in the next chapter.

![Initiate Instance Launch](../../Assets/4_Launch_Instance/4_LaunchTemplate_Launch.png)

![Launch Template Summary](../../Assets/4_Launch_Instance/4_LaunchTemplate_Trigger.png)

The result should be that you have multiple EC2 instances launched and running, with the correct network and storage configuration for multi-node training workloads. You can verify this by going to the EC2 console and checking the instances that you launched using the launch template. You should see that they are in the "running" state, and that they have the correct instance type, network configuration, and storage setup as defined in the launch template.

![Launched Instances](../../Assets/4_Launch_Instance/4_EC2_Instances.png)

**Step 2:** After launching the instances, navigate to the "Elastic IPs" section in the EC2 console and allocate new Elastic IP addresses. Allocate one Elastic IP address for each instance launched in the previous step.

![Allocate Elastic IP](../../Assets/4_Launch_Instance/4_ElasticIP_Allocate.png)

**Step 3:** After allocating the Elastic IP addresses, associate each one with the corresponding EC2 instance. Select the Elastic IP address you want to associate, click "Actions", and then click "Associate Elastic IP address".

Note that the usual practice of associating Elastic IP addresses directly with EC2 instances only works when the instance has a single ENA-enabled network card. In our setup, we have multiple network cards with only one being ENA-enabled, so we need to associate the Elastic IP address with the correct network card (i.e., the ENA-enabled one) rather than the instance itself.

To find the ENA-enabled network card, click on the instance ID in the EC2 console, navigate to the "Networking" tab, and identify the ENA-enabled card. It will have device index 0 (as configured in the launch template) and will be the only network card with a private IP address assigned. Select this interface ID when associating the public Elastic IP address.

![Query ENA-enabled Network Card](../../Assets/4_Launch_Instance/4_ENA_NetworkCard.png)

![Associate Elastic IP](../../Assets/4_Launch_Instance/4_Associate_PublicIP.png)

It is worth noting that a public-IP-per-instance setup is not strictly necessary for multi-node training. In larger teams and production setups, instances are typically launched in a private subnet and accessed through a bastion host (commonly known as a head node). In such setups, only the bastion host (typically a CPU instance) has a public IP address, while the training instances remain in a private subnet. This is a more secure and cost-effective approach, as it limits exposure to the public internet and provides better access control. However, for simplicity in this tutorial, we assign public IP addresses to enable direct SSH access. For alternative infrastructure setups, refer to the documentation on [AWS ParallelCluster](https://github.com/awslabs/awsome-distributed-training/tree/main/1.architectures/2.aws-parallelcluster) and [SageMaker HyperPod](https://github.com/awslabs/awsome-distributed-training/tree/main/1.architectures/5.sagemaker-hyperpod).
