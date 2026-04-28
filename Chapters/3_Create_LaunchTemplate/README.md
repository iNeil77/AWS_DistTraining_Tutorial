# Create Optimized EC2 Launch Template for Multi-node Training

In this chapter, we will create an EC2 launch template optimized for multi-node training workloads. This involves configuring the template to maximize network bandwidth, creating a swap partition on the NVMe instance storage (useful when loading large models and datasets), and mounting the FSx for Lustre storage. By creating a launch template with these optimizations, any instances launched from it will be configured correctly and repeatably for our training workloads.

## Steps

**Step 1:** Create a new EC2 launch template. Go to the EC2 console, click on "Launch Templates" in the left-hand menu, and then click on "Create launch template". Provide a name and description, and select the AMI you created in the previous chapter (or any other AMI you prefer). Choose an instance type suitable for your training workloads (e.g., p4d or p5 for large-scale training).

![Create Launch Template](../../Assets/3_Create_LaunchTemplate/3_LaunchTemplate_Starter.png)

**Step 2:** Configure the correct high-level subnet and security group settings for the launch template based on what we configured in the previous chapters. This is important to ensure that the instances launched using this template can communicate with each other and with other AWS services as needed.

![Configure Subnet and Security Group](../../Assets/3_Create_LaunchTemplate/3_LaunchTemplate_NetworkBasic.png)

**Step 3:** In the "Advanced network configuration" section inside network settings, add settings for multiple network cards to maximize network bandwidth for multi-node training and communication with the FSx for Lustre storage. This is an involved step where many bespoke configurations are possible, but for simplicity we will add 4 network cards in total: the first configured with both EFA and ENA, and the remaining 3 configured with EFA only. Make sure the security groups and subnet you select are consistent with those created in the previous chapters.

In this setup only the first network card can be assigned an IP address, and the other 3 network cards will be used for EFA communication only. This is a common setup for multi-node training workloads where the first network card is used for general communication and the other network cards are used for high-speed communication between instances using EFA. For other possible setups and configurations, please refer to the AWS documentation on [Maximizing EFA bandwidth](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/efa-acc-inst-types.html).

![Configure Network Cards](../../Assets/3_Create_LaunchTemplate/3_LaunchTemplate_NetworkAdvanced1.png)

![Configure Network Cards](../../Assets/3_Create_LaunchTemplate/3_LaunchTemplate_NetworkAdvanced2.png)

**Step 4:** In the "Storage (volumes)" section, configure the EBS volumes for the launch template. Make sure to set the bandwidth and IOPS to appropriate levels for your training workloads. You can add additional EBS volumes if needed for training data or model checkpoints. However, our setup mainly relies on FSx for Lustre storage, so we will not add additional EBS volumes in this step.

Note the "Instance store (ephemeral)" section in the storage configuration. This allows you to configure the instance store volumes that are physically attached to the instance. For certain instance types (e.g., p4d and p5), these are NVMe drives that provide very high bandwidth and low-latency storage, which can be beneficial for workloads requiring fast data access. They can be used as scratch storage where intermediately processed versions of the training data are stored for fast access during training.

We will leverage this fast storage as a swap partition, which is beneficial when loading large models and datasets that may not fit entirely in memory. We create a swap file on the instance store volumes and configure the system to use it as swap space during training, helping prevent out-of-memory errors.

![Configure Storage](../../Assets/3_Create_LaunchTemplate/3_LaunchTemplate_Storage.png)

**Step 5:** In the "Advanced details" section, setup the cluster placement group settings for the launch template. This is important to ensure that the instances launched using this template are placed in the same cluster placement group, which can help improve network performance for multi-node training workloads. Select the cluster placement group that you created in the previous chapters.

![Configure Cluster Placement Group](../../Assets/3_Create_LaunchTemplate/3_LaunchTemplate_Placement.png)

**Step 6:** In the "Advanced details" section, add user data scripts to the launch template to automate instance setup at launch time. This includes commands to create a swap file on the instance store volumes and configure it as swap space, as well as commands to mount the FSx for Lustre storage.

![Configure User Data](../../Assets/3_Create_LaunchTemplate/3_LaunchTemplate_UserScript.png)

Below is our example of a user data script that performs these tasks. You can customize this script as needed for your specific training workloads and setup.

```bash

#!/bin/bash

sudo mkdir -p /mnt/fsx
sudo mount -t lustre -o relatime,flock fs-012cdc904f134137f.fsx.us-west-2.amazonaws.com@tcp:/d3mlnb4v /mnt/fsx
sudo chmod -R 777 /mnt/fsx

sudo fallocate -l 8192G /opt/dlami/nvme/swapfile
sudo chmod 600 /opt/dlami/nvme/swapfile
sudo mkswap /opt/dlami/nvme/swapfile
sudo swapon /opt/dlami/nvme/swapfile
sudo sysctl vm.swappiness=1
sudo mkdir -p /opt/dlami/nvme/shm
sudo chmod 777 /opt/dlami/nvme/shm
```
