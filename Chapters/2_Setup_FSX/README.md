# Setup FSx for Lustre Storage

This chapter will guide you through the process of setting up FSx for Lustre storage on AWS. FSx for Lustre is a high-performance file system optimized for compute-intensive workloads, making it an ideal shared storage solution for distributed training. By following the steps outlined in this chapter, you will create and configure an FSx for Lustre file system that your EC2 instances can use for storing and accessing training data and model checkpoints via the EFA network interface.

## Steps

**Step 1:** Open the FSx console and click on "Create file system". Select "Lustre" as the file system type and choose the appropriate configuration options based on your requirements. This includes selecting the storage capacity, performance mode, and throughput capacity. You can also choose to enable encryption if desired.

![Create FSX File System](../../Assets/2_Setup_FSX/2_FSX_Pane1.png)

**Step 2:** In the configuration options, make sure to select the same VPC and availability zone as your EC2 instances so they can access the file system. Select the EFA option for the network interface, which allows your instances to access the FSx file system using EFA for high-performance data transfer.

Also select the security group you created in the previous chapter, which has the appropriate rules to allow communication between the EC2 instances and the FSx file system.

![FSX Configuration](../../Assets/2_Setup_FSX/2_FSX_Pane2.png)

The remaining configuration options can be set based on your specific requirements. For example, you can choose the storage capacity and performance mode based on the size of your training data and the expected I/O workload. You can also enable encryption to ensure your data is secure. These options can be left at their default values if they are sufficient for your needs.
