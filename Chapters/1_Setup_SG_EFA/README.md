# Setup Security Groups and Placement Groups for multi-node training

This chapter will guide you through the process of setting up security groups and cluster placement groups for your EC2 instances. Security groups act as virtual firewalls that control the inbound and outbound traffic to your instances, while cluster placement groups help ensure that your instances are placed in a way that optimizes network performance for distributed training.

Subsequent chapters will cover modifying EC2 launch templates to enable EFA and maximize network bandwidth, but this chapter focuses on the security group and placement group setup itself.

## Steps

**Step 1:** Open the EC2 console and navigate to the "Security Groups" section. Click on "Create Security Group" to create a new security group for your instances. Create a basic configuration allowing inbound SSH access (port 22) and HTTP access (port 80) while enabling all outbound traffic. You can customize the security group rules based on your specific requirements.

![Create Security Group](../../Assets/1_Setup_SG_PG/1_SG_Pane1.png)

**Step 2:** With the security group created, we now set up self-referential rules that allow the EC2 instances to communicate with each other. This is essential for distributed training, as the instances need to exchange data and coordinate their actions. Edit the inbound rules of the security group and add a new rule that allows all traffic (or specific ports if you prefer) from the security group itself. Any instance associated with this security group will then be able to communicate with any other instance in the same group.

Select "Edit inbound rules" for the security group we just created, and add a new rule allowing all traffic from the security group itself. Choose "Custom" for the type, "All traffic" for the protocol, and then select the security group from the "Source" dropdown.

![Edit Inbound Rules](../../Assets/1_Setup_SG_PG/1_SG_Inbound.png)

![Add Self-Referential Rule](../../Assets/1_Setup_SG_PG/1_SG_Inbound_SelfRef.png)

Similarly, edit the outbound rules of the security group to allow all traffic to the security group itself. Select "Edit outbound rules" and add a new rule choosing "Custom" for the type, "All traffic" for the protocol, and then selecting the security group from the "Destination" dropdown.

![Edit Outbound Rules](../../Assets/1_Setup_SG_PG/1_SG_Outbound.png)

![Add Self-Referential Rule](../../Assets/1_Setup_SG_PG/1_SG_Outbound_SelfRef.png)

**Step 3:** With the security group set up, we can now create a cluster placement group. Cluster placement groups are a logical grouping of instances within a single Availability Zone that enables applications to participate in a low-latency, high-bandwidth network. This is particularly beneficial for distributed training workloads, as it can significantly improve the communication performance between instances.

Placement groups are versatile and can be used with different instance types. They can also be leveraged to improve resilience of deployed fleets by spreading instances across different underlying hardware groupings. However, for this tutorial we will create a cluster placement group, which is specifically designed to optimize network performance for distributed training workloads. Navigate to the "Placement Groups" section in the EC2 console and click on "Create Placement Group". Provide a name for the placement group and select "Cluster".

![Create Placement Group](../../Assets/1_Setup_SG_PG/1_PG_Pane1.png)
