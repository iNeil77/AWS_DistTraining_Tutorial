# Setup and Test EC2 Instance and Create Final EC2 AMI (Optional)

This chapter will guide you through the process of setting up and testing a test EC2 instance, which we will initialize with the Deep Learning AMI (DLAMI). The DLAMI is a pre-built Amazon Machine Image that comes with popular deep learning frameworks and tools pre-installed, along with important conveniences such as NVIDIA drivers, CUDA, cuDNN, and the GPU-enabled Docker runtime. We will use this as a springboard to create our own custom AMI that contains important dependencies such as enroot.

For more information on the DLAMI, please refer to the [DLAMI documentation](https://docs.aws.amazon.com/dlami/latest/devguide/appendix-ami-release-notes.html). Additionally, a more programmatic way of creating custom AMIs is detailed in the documentation, which can be found [here](https://github.com/awslabs/awsome-distributed-training/tree/main/2.ami_and_containers/1.amazon_machine_image).

## Steps

**Step 1:** Open the EC2 console and launch an instance with the DLAMI. You can find it by searching for "Deep Learning AMI" in the AWS Marketplace when launching an instance. Choose the appropriate version based on your needs (e.g., Ubuntu, PyTorch, TensorFlow). We will use the Ubuntu version for this tutorial.

![Create EC2 Instance](../../Assets/0_Setup_TestEC2/0_EC2_Pane1.png)

![EC2 Instance List](../../Assets/0_Setup_TestEC2/0_EC2_Pane2.png)

**Step 2:** Install necessary dependencies and tools on the EC2 instance. This includes enroot and gh cli. You can follow the installation instructions for each of these tools from their respective documentation.

```bash
# Update package lists and existing packages
sudo apt-get update && sudo apt-get upgrade -y
```

```bash
# Install gh cli
(type -p wget >/dev/null || (sudo apt update && sudo apt install wget -y)) \
     && sudo mkdir -p -m 755 /etc/apt/keyrings \
     && out=$(mktemp) && wget -nv -O$out https://cli.github.com/packages/githubcli-archive-keyring.gpg \
     && cat $out | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
     && sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
     && sudo mkdir -p -m 755 /etc/apt/sources.list.d \
     && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
     && sudo apt update \
     && sudo apt install gh -y
```

```bash
# Install enroot
arch=$(dpkg --print-architecture)
curl -fSsL -O https://github.com/NVIDIA/enroot/releases/download/v4.1.1/enroot_4.1.1-1_${arch}.deb
curl -fSsL -O https://github.com/NVIDIA/enroot/releases/download/v4.1.1/enroot+caps_4.1.1-1_${arch}.deb # optional
sudo apt install -y ./*.deb
```

```bash
# Install mamba3 and create a specced out environment (optional, useful for debugging and testing out dependencies before baking them into the final AMI)
curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
chmod +x Miniforge3-$(uname)-$(uname -m).sh
./Miniforge3-$(uname)-$(uname -m).sh -b -p $HOME/mamba3
```

```bash
# Create Admin mamba environment (optional)
source $HOME/mamba3/bin/activate
mamba create -n Admin -y python=3.12 ipython
pip install boto3 botocore awscli \
     einops \
     hatchling \
     hjson\
     msgpack \
     packaging \
     ninja \
     uv
```

**Step 3:** Build the Docker images for training and evaluating the reward model. You can find the Dockerfile for the reward model training [here](../../Environment/Axolotl.Dockerfile). The image is instrumented with the necessary dependencies for training and evaluating the reward model, including the Axolotl framework and the reward-bench evaluation suite. Navigate to the directory containing the Dockerfile and build the image using the following command:

```bash
# Navigate to the Environment directory containing the Dockerfile
cd ../../Environment

# Build the docker image for reward model training
docker build -t username/reward_model_training:latest -f Axolotl.Dockerfile .
```

**Step 4:** Clean up the instance and create a custom AMI. After installing the necessary dependencies and tools, you can create a custom AMI from the instance. This allows you to launch new instances with all the dependencies pre-installed, saving time and effort in the future. To create an AMI, go to the EC2 console, select the instance, and click on "Actions" > "Image and Templates" > "Create Image". Provide a name and description for the AMI and click "Create Image". Once the AMI is created, you can use it to launch new instances with the same configuration. Note that creating an AMI is optional, but it can be beneficial for streamlining the setup process in future chapters. If you choose not to create an AMI, you can simply repeat the installation steps on any new instances you launch.

![Create AMI](../../Assets/0_Setup_TestEC2/0_EC2_Pane3.png)

![Create AMI](../../Assets/0_Setup_TestEC2/0_EC2_Pane4.png)

Make sure to clean up the instance before creating the AMI by removing any unnecessary files or data generated during setup. This helps reduce the AMI size and ensures it only contains the necessary dependencies. You can use `sudo apt-get clean` to remove cached package files and `rm -rf /path/to/unnecessary/files` to remove anything else that is not needed. Purging the Docker cache with `docker system prune -a` will remove all unused images, containers, and networks. Similarly, clean up any dot folders in the home directory that may contain cached data (e.g., `rm -rf ~/.cache` and `rm -rf ~/.config`). By cleaning up before creating the AMI, you can ensure the resulting image is as lean as possible.
