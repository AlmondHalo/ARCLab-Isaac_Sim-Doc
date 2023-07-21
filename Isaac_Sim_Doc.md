## Omniverse/Isaac Sim Documentation

The following is meant to serve as an update to the ARCLab server's [Docker Guide](https://github.com/ucsdarclab/ServerDockerGuidance/blob/main/README.md?plain=1) describing the installation process of **NVIDIA Omniverse/Isaac Sim** and one method of creating a containerized environment to run Isaac Sim off the lab's server.

Same as with the Docker Guide, there are other techniques/methods which may be a good extension of this document or may be more efficient. Feel free to update this page with the prevalent information.

### Prerequisite(s):
Complete the necessary environment setup on the [Docker Guide](https://github.com/ucsdarclab/ServerDockerGuidance/blob/main/README.md?plain=1).

Summary:
1. Install Docker
2. Create a work environment within the lab's server
### On Local Machine: [(NVIDIA Installation Documentation)](https://docs.omniverse.nvidia.com/isaacsim/latest/install_workstation.html)

#### Installation: Omniverse Launcher, Omniverse Streaming Client
The **Omniverse Launcher** can be used to launch Isaac Sim directly (assuming Isaac Sim is to be launched from your local maching and ***NOT*** from a server). It is also needed to download and launch the **Omniverse Streaming Client** which will later be used to stream from the remote container to your local machine.
1. Download the [Omniverse Launcher](https://www.nvidia.com/en-us/omniverse/download/)
    * This will require you to create your own NVIDIA account if you have not done so already.
    * Once installation is complete, open the Omniverse Launcher application, complete the installation process, and log in using your NVIDIA account information.
2. Install the [Omniverse Streaming Client](https://docs.omniverse.nvidia.com/streaming-client/104.0.0/user-manual.html)
    * From the ***Exchange*** tab, search "Omniverse Streaming Client", select, and install the application.
    * Launch the application from the ***Library*** tab.

### On Remote Machine:

#### Installation: NVIDIA Dockerfile(s)
In substitute of downloading **Isaac Sim** locally, the application may be launched via Dockerfile in its own container.
1. Follow the instructions on the [Isaac Sim github](https://github.com/NVIDIA-Omniverse/IsaacSim-dockerfiles) to get access to clone the repository. 
    * Get access to the [Isaac Sim Container](https://catalog.ngc.nvidia.com/orgs/nvidia/containers/isaac-sim), and either generate an [NGC API Key](https://docs.nvidia.com/ngc/gpu-cloud/ngc-user-guide/index.html#generating-api-key) or reference a currently existing one (assuming you've generated one previously)
2. Log onto the server and navigate the path to wherever you choose to clone the repository.
    * Cloning the Isaac Sim repository requires access. <br/>Run:<br/>
        ```docker login nvcr.io```<br/>
    Use **$oauthtoken** in place of username <br/>
    Use your generated **NGC API key** as the password (copy and paste) <br/>
    * Clone the [Isaac Sim Container](https://catalog.ngc.nvidia.com/orgs/nvidia/containers/isaac-sim)
    * Build the image (the original image has been deleted from the ARCLab server, check that it has not been recreated before building):<br/>
    ```
    docker build --pull -t \
        isaac-sim:2022.2.1-ubuntu20.04 \
        --build-arg ISAACSIM_VERSION=2022.2.1 \
        --build-arg BASE_DIST=ubuntu20.04 \
        --build-arg CUDA_VERSION=11.4.2 \
        --build-arg VULKAN_SDK_VERSION=1.3.224.1 \
        --file Dockerfile.2022.2.1-ubuntu20.04 
    ```
3. Create a ```.sh``` file, paste the following within the file:
    ```
    docker run --name isaac-sim --entrypoint bash -it --gpus 1 -e "ACCEPT_EULA=Y" --rm --network=host \
        -v ~/docker/isaac-sim/cache/kit:/isaac-sim/kit/cache/Kit:rw \
        -v ~/docker/isaac-sim/cache/ov:/root/.cache/ov:rw \
        -v ~/docker/isaac-sim/cache/pip:/root/.cache/pip:rw \
        -v ~/docker/isaac-sim/cache/glcache:/root/.cache/nvidia/GLCache:rw \
        -v ~/docker/isaac-sim/cache/computecache:/root/.nv/ComputeCache:rw \
        -v ~/docker/isaac-sim/logs:/root/.nvidia-omniverse/logs:rw \
        -v ~/docker/isaac-sim/data:/root/.local/share/ov/data:rw \
        -v ~/docker/isaac-sim/documents:/root/Documents:rw \
        isaac-sim:2022.2.1-ubuntu20.04
    ```
    * Running the ```.sh``` file will bring up a bash terminal with all the associated files necessary for running Isaac Sim.
4. In the /isaac-sim/ directory, run in headless mode:
    ```
    ./runheadless.native.sh
    ```