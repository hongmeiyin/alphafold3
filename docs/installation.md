# Installation and Running Your First Prediction 安装并运行您的第一个预测

You will need a machine running Linux; AlphaFold 3 does not support other
operating systems. Full installation requires up to 1 TB of disk space to keep
genetic databases (SSD storage is recommended) and an NVIDIA GPU with Compute
Capability 8.0 or greater (GPUs with more memory can predict larger protein
structures). We have verified that inputs with up to 5,120 tokens can fit on a
single NVIDIA A100 80 GB, or a single NVIDIA H100 80 GB. We have verified
numerical accuracy on both NVIDIA A100 and H100 GPUs.
你需要一台运行Linux的机器；AlphaFold 3不支持其他操作系统。完整安装需要高达1TB的磁盘空间来保存遗传数据库（建议使用SSD存储）和具有8.0或更高计算能力的NVIDIA GPU（具有更多内存的GPU可以预测更大的蛋白质结构）。我们已经验证，最多5120个令牌的输入可以放在单个NVIDIA A100 80 GB或单个NVIDIA H100 80 GB上。我们已经在NVIDIA A100和H100 GPU上验证了数值精度。

Especially for long targets, the genetic search stage can consume a lot of RAM –
we recommend running with at least 64 GB of RAM.
特别是对于长目标，基因搜索阶段可能会消耗大量RAM——我们建议使用至少64 GB的RAM运行。

We provide installation instructions for a machine with an NVIDIA A100 80 GB GPU
and a clean Ubuntu 22.04 LTS installation, and expect that these instructions
should aid others with different setups. If you are installing locally outside
of a Docker container, please ensure CUDA, cuDNN, and JAX are correctly
installed; the
[JAX installation documentation](https://jax.readthedocs.io/en/latest/installation.html#nvidia-gpu)
is a useful reference for this case. Please note that the Docker container
requires that the host machine has CUDA 12.6 installed.
我们为配备NVIDIA A100 80 GB GPU和干净的Ubuntu 22.04 LTS安装的机器提供安装说明，并希望这些说明能帮助其他人进行不同的设置。如果您在Docker容器之外进行本地安装，请确保CUDA、cuDNN和JAX已正确安装；[JAX安装文档](https://jax.readthedocs.io/en/latest/installation.html#nvidia-gpu）对于这种情况是一个有用的参考。请注意，Docker容器要求主机安装CUDA 12.6。

The instructions provided below describe how to:
下面提供的说明描述了如何：

1.  Provision a machine on GCP. 在GCP上提供一台机器。
1.  Install Docker. 安装Docker。
1.  Install NVIDIA drivers for an A100. 为A100安装NVIDIA驱动程序。
1.  Obtain genetic databases. 获取基因数据库。
1.  Obtain model parameters. 获取模型参数。
1.  Build the AlphaFold 3 Docker container or Singularity image. 构建AlphaFold 3 Docker容器或Singularity镜像。

## Provisioning a Machine 配置机器

Clean Ubuntu images are available on Google Cloud, AWS, Azure, and other major
platforms.
干净的Ubuntu映像可在Google Cloud、AWS、Azure和其他主要平台上使用。

Using an existing Google Cloud project, we provisioned a new machine:
使用现有的Google Cloud项目，我们配置了一台新机器：
*   We recommend using `--machine-type a2-ultragpu-1g` but feel free to use
    `--machine-type a2-highgpu-1g` for smaller predictions.
    我们建议使用`--machine type a2-ultrapu-1g`，但对于较小的预测，可以随意使用`--Machine-type a2-highgpu-1g`
*   If desired, replace `--zone us-central1-a` with a zone that has quota for
    the machine you have selected. See
    [gpu-regions-zones](https://cloud.google.com/compute/docs/gpus/gpu-regions-zones).
    如果需要，请将`--zone us-central1-a`替换为具有所选机器配额的区域。请参见[gpu区域](https://cloud.google.com/compute/docs/gpus/gpu-regions-zones)

```sh
gcloud compute instances create alphafold3 \
    --machine-type a2-ultragpu-1g \
    --zone us-central1-a \
    --image-family ubuntu-2204-lts \
    --image-project ubuntu-os-cloud \
    --maintenance-policy TERMINATE \
    --boot-disk-size 1000 \
    --boot-disk-type pd-balanced
```

This provisions a bare Ubuntu 22.04 LTS image on an
[A2 Ultra](https://cloud.google.com/compute/docs/accelerator-optimized-machines#a2-vms)
machine with 12 CPUs, 170 GB RAM, 1 TB disk and NVIDIA A100 80 GB GPU attached.
We verified the following installation steps from this point.
这在[A2 Ultra]上提供了一个裸露的Ubuntu 22.04 LTS映像(https://cloud.google.com/compute/docs/accelerator-optimized-machines#a2-vms）机器，配备12个CPU、170 GB RAM、1 TB磁盘和连接的NVIDIA A100 80 GB GPU。我们从这一点开始验证了以下安装步骤。

## Installing Docker 安装Docker

These instructions are for rootless Docker.
此说明适用于无根（rootless）Docker模式

### Installing Docker on Host 在主机上安装Docker

Note these instructions only apply to Ubuntu 22.04 LTS images, see above.
请注意，这些说明仅适用于Ubuntu 22.04 LTS映像，见上文。

Add Docker's official GPG key. Official Docker instructions are
[here](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository).
The commands we ran are:
添加Docker的官方GPG密钥。Docker的官方说明在[这里](https://docs.docker.com/engine/install/ubuntu/#install-使用存储库）。我们运行的命令是：

```sh
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Add the repository to apt sources:
将存储库添加到apt-sources：

```sh
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo docker run hello-world
```

### Enabling Rootless Docker 启用无根Docker

Official Docker instructions are
[here](https://docs.docker.com/engine/security/rootless/#distribution-specific-hint).
The commands we ran are:
Docker的官方说明在[这里](https://docs.docker.com/engine/security/rootless/#distribution-具体提示）。我们运行的命令是：

```sh
sudo apt-get install -y uidmap systemd-container

sudo machinectl shell $(whoami)@ /bin/bash -c 'dockerd-rootless-setuptool.sh install && sudo loginctl enable-linger $(whoami) && DOCKER_HOST=unix:///run/user/1001/docker.sock docker context use rootless'
```

## Installing GPU Support 安装GPU支持

### Installing NVIDIA Drivers 安装NVIDIA驱动程序

Official Ubuntu instructions are
[here](https://documentation.ubuntu.com/server/how-to/graphics/install-nvidia-drivers/).
The commands we ran are:
Ubuntu官方说明在[这里](https://documentation.ubuntu.com/server/how-to/graphics/install-nvidia-drivers/).我们运行的命令是：

```sh
sudo apt-get -y install alsa-utils ubuntu-drivers-common
sudo ubuntu-drivers install

sudo nvidia-smi --gpu-reset

nvidia-smi  # Check that the drivers are installed.
```

Accept the "Pending kernel upgrade" dialog if it appears.
如果出现"等待内核升级"对话框，请接受。

You will need to reboot the instance with `sudo reboot now` to reset the GPU if
you see the following warning:
如果您看到以下警告，则需要使用"sudo reboot now"重新启动实例以重置GPU：

```text
NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver.
Make sure that the latest NVIDIA driver is installed and running.
```
NVIDIA-SMI失败，因为它无法与NVIDIA驱动程序通信。确保已安装并运行最新的NVIDIA驱动程序。

Proceed only if `nvidia-smi` has a sensible output.
仅当"nvidia-smi"有合理的输出时才继续。

### Installing NVIDIA Support for Docker 为Docker安装NVIDIA支持

Official NVIDIA instructions are
[here](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html).
The commands we ran are:
NVIDIA官方说明[此处](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html).我们运行的命令是：

```sh
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
nvidia-ctk runtime configure --runtime=docker --config=$HOME/.config/docker/daemon.json
systemctl --user restart docker
sudo nvidia-ctk config --set nvidia-container-cli.no-cgroups --in-place
```

Check that your container can see the GPU:
检查你的容器是否可以看到GPU：

```sh
docker run --rm --gpus all nvidia/cuda:12.6.0-base-ubuntu22.04 nvidia-smi
```

Example output:
输出示例：

```text
Mon Nov  11 12:00:00 2024
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 550.120                Driver Version: 550.120        CUDA Version: 12.6     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA A100-SXM4-80GB          Off |   00000000:00:05.0 Off |                    0 |
| N/A   34C    P0             51W /  400W |       1MiB /  81920MiB |      0%      Default |
|                                         |                        |             Disabled |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|  No running processes found                                                             |
+-----------------------------------------------------------------------------------------+
```

## Obtaining AlphaFold 3 Source Code 获取AlphaFold 3源代码

Install `git` and download the AlphaFold 3 repository:
安装`git`并下载AlphaFold 3存储库：

```sh
git clone https://github.com/google-deepmind/alphafold3.git
```

## Obtaining Genetic Databases 获取基因数据库

This step requires `wget` and `zstd` to be installed on your machine. On
Debian-based systems install them by running `sudo apt install wget zstd`.
此步骤要求在您的计算机上安装`wget`和`zstd`。在基于Debian的系统上，通过运行`sudo apt install wget zstd`来安装它们。

AlphaFold 3 needs multiple genetic (sequence) protein and RNA databases to run:

*   [BFD small](https://bfd.mmseqs.com/)
*   [MGnify](https://www.ebi.ac.uk/metagenomics/)
*   [PDB](https://www.rcsb.org/) (structures in the mmCIF format)
*   [PDB seqres](https://www.rcsb.org/)
*   [UniProt](https://www.uniprot.org/uniprot/)
*   [UniRef90](https://www.uniprot.org/help/uniref)
*   [NT](https://www.ncbi.nlm.nih.gov/nucleotide/)
*   [RFam](https://rfam.org/)
*   [RNACentral](https://rnacentral.org/)

We provide a bash script `fetch_databases.sh` that can be used to download and
set up all of these databases. This process takes around 45 minutes when not
installing on local SSD. We recommend running the following in a `screen` or
`tmux` session as downloading and decompressing the databases takes some time.
我们提供了一个bash脚本`fetch_databases.sh`，可用于下载和设置所有这些数据库。如果不在本地SSD上安装，此过程大约需要45分钟。我们建议在`screen`或`tmux`会话中运行以下操作，因为下载和解压缩数据库需要一些时间。

```sh
cd alphafold3  # Navigate to the directory with cloned AlphaFold 3 repository.
./fetch_databases.sh [<DB_DIR>]
```

This script downloads the databases from a mirror hosted on GCS, with all
versions being the same as used in the AlphaFold 3 paper, to the directory
`<DB_DIR>`. If not specified, the default `<DB_DIR>` is
`$HOME/public_databases`.
此脚本将数据库从GCS上托管的镜像下载到目录`<DB_DIR>`，所有版本都与AlphaFold 3论文中使用的版本相同。如果没有指定，默认的`<DB_DIR>`是`$HOME/public_databases`。

:ledger: **Note: The download directory `<DB_DIR>` should *not* be a
subdirectory in the AlphaFold 3 repository directory.** If it is, the Docker
build will be slow as the large databases will be copied during the image
creation.
:ledger: 注意：下载目录 <DB_DIR> 不应是 AlphaFold 3 代码库目录的子目录。 否则，Docker 镜像构建过程会非常缓慢，因为在创建镜像时会复制庞大的数据库文件。

:ledger: **Note: The total download size for the full databases is around 252 GB
and the total size when unzipped is 630 GB. Please make sure you have sufficient
hard drive space, bandwidth, and time to download. We recommend using an SSD for
better genetic search performance.**
:ledger: 注意：完整数据库的总下载大小约为 252 GB，解压后总大小为 630 GB。请确保您有足够的硬盘空间、带宽和下载时间。我们建议使用 SSD 硬盘以获得更好的遗传搜索性能。

:ledger: **Note: If the download directory and datasets don't have full read and
write permissions, it can cause errors with the MSA tools, with opaque
(external) error messages. Please ensure the required permissions are applied,
e.g. with the `sudo chmod 755 --recursive <DB_DIR>` command.**
:ledger: 注意：如果下载目录和数据集没有完整的读写权限，可能会导致 MSA 工具出现错误，并伴随难以理解的（外部）错误信息。请确保应用了必要的权限，例如使用 sudo chmod 755 --recursive <DB_DIR> 命令。
Once the script has finished, you should have the following directory structure:
脚本完成后，您应该具有以下目录结构：

```sh
mmcif_files/  # Directory containing ~200k PDB mmCIF files. 包含约200k PDB mmCIF文件的目录
bfd-first_non_consensus_sequences.fasta
mgy_clusters_2022_05.fa
nt_rna_2023_02_23_clust_seq_id_90_cov_80_rep_seq.fasta
pdb_seqres_2022_09_28.fasta
rfam_14_9_clust_seq_id_90_cov_80_rep_seq.fasta
rnacentral_active_seq_id_90_cov_80_linclust.fasta
uniprot_all_2021_04.fa
uniref90_2022_05.fa
```

Optionally, after the script finishes, you may want copy databases to an SSD.或者，在脚本完成后，您可能需要将数据库复制到SSD。
You can use theses two scripts:您可以使用这两个脚本：

*   `src/scripts/gcp_mount_ssd.sh [<SSD_MOUNT_PATH>]` Mounts and formats an
    unmounted GCP SSD drive to the specified path. It will skip the either step
    if the disk is either already formatted or already mounted. The default
    `<SSD_MOUNT_PATH>` is `/mnt/disks/ssd`.
    src/scripts/gcp_mount_ssd.sh [<SSD_MOUNT_PATH>]：将未挂载的GCP SSD驱动器挂载并格式化到指定路径。如果磁盘已经格式化或已经挂载，则会跳过相应步骤。默认的<SSD_MOUNT_PATH>为/mnt/disks/ssd
*   `src/scripts/copy_to_ssd.sh [<DB_DIR>] [<SSD_DB_DIR>]` this will copy as
    many files that it can fit on to the SSD. The default `<DB_DIR>` is
    `$HOME/public_databases`, and must match the path used in the
    `fetch_databases.sh` command above, and the default `<SSD_DB_DIR>` is
    `/mnt/disks/ssd/public_databases`.
    src/scripts/copy_to_ssd.sh [<DB_DIR>] [<SSD_DB_DIR>]：此脚本将尽可能多的文件复制到SSD上。默认的<DB_DIR>为$HOME/public_databases，且必须与上述fetch_databases.sh命令使用的路径匹配；默认的<SSD_DB_DIR>为/mnt/disks/ssd/public_databases

## Obtaining Model Parameters 获取模型参数

To request access to the AlphaFold 3 model parameters, please complete
[this form](https://forms.gle/svvpY4u2jsHEwWYS6). Access will be granted at
Google DeepMind’s sole discretion. We will aim to respond to requests within 2–3
business days. You may only use AlphaFold 3 model parameters if received
directly from Google. Use is subject to these
[terms of use](https://github.com/google-deepmind/alphafold3/blob/main/WEIGHTS_TERMS_OF_USE.md).
要请求访问AlphaFold 3模型参数，请填写[此表](https://forms.gle/svvpY4u2jsHEwWYS6).访问权限将由Google DeepMind全权决定。我们的目标是在2-3个工作日内回复请求。如果直接从谷歌收到，您只能使用AlphaFold 3模型参数。使用受这些[使用条款]的约束(https://github.com/google-deepmind/alphafold3/blob/main/WEIGHTS_TERMS_OF_USE.md).

Once access has been granted, download the model parameters to a directory of
your choosing, referred to as `<MODEL_PARAMETERS_DIR>` in the following
instructions. As with the databases, this should *not* be a subdirectory in the
AlphaFold 3 repository directory.
授予访问权限后，将模型参数下载到您选择的目录中，在以下说明中称为`<model_parameters_DIR>`。与数据库一样，这不应该是AlphaFold 3存储库目录中的子目录。

## Building the Docker Container That Will Run AlphaFold 3 构建运行AlphaFold 3的Docker容器

Then, build the Docker container. This builds a container with all the right
python dependencies:
然后，构建Docker容器。这将构建一个包含所有正确python依赖项的容器：

```sh
docker build -t alphafold3 -f docker/Dockerfile .
```

Create an input JSON file, using either the example in the
[README](https://github.com/google-deepmind/alphafold3?tab=readme-ov-file#installation-and-running-your-first-prediction)
or a
[custom input](https://github.com/google-deepmind/alphafold3/blob/main/docs/input.md),
and place it in a directory, e.g. `$HOME/af_input`. You can now run AlphaFold 3!

```sh
docker run -it \
    --volume $HOME/af_input:/root/af_input \
    --volume $HOME/af_output:/root/af_output \
    --volume <MODEL_PARAMETERS_DIR>:/root/models \
    --volume <DB_DIR>:/root/public_databases \
    --gpus all \
    alphafold3 \
    python run_alphafold.py \
    --json_path=/root/af_input/fold_input.json \
    --model_dir=/root/models \
    --output_dir=/root/af_output
```

where `$HOME/af_input` is the directory containing the input JSON file;
`$HOME/af_output` is the directory where the output will be written to; and
`<DB_DIR>` and `<MODEL_PARAMETERS_DIR>` are the directories containing the
databases and model parameters. The values of these directories must match the
directories used in previous steps for downloading databases and model weights,
and for the input file.
其中`$HOME/af_input`是包含输入JSON文件的目录；`$HOME/af_output`是输出将写入的目录；`<DB_DIR>`和`<MODEL_PARAMETERS_DIR>`是包含数据库和模型参数的目录。这些目录的值必须与前面步骤中用于下载数据库和模型权重以及输入文件的目录相匹配。

:ledger: Note: You may also need to create the output directory,
`$HOME/af_output` directory before running the `docker` command and make it and
the input directory writable from the docker container, e.g. by running `chmod
755 $HOME/af_input $HOME/af_output`. In most cases `docker` and
`run_alphafold.py` will create the output directory if it does not exist.
:ledger: 注意：在运行 docker 命令前，您可能需要先创建输出目录 $HOME/af_output，并确保该目录和输入目录对 Docker 容器可写，例如通过运行 chmod 755 $HOME/af_input $HOME/af_output 命令。在大多数情况下，如果输出目录不存在，docker 和 run_alphafold.py 会自动创建它。

:ledger: **Note: In the example above the databases have been placed on the
persistent disk, which is slow.** If you want better genetic and template search
performance, make sure all databases are placed on a local SSD.
:ledger: 注意：上述示例中将数据库放在了持久化磁盘上，其速度较慢。 如果您希望获得更好的遗传搜索和模板搜索性能，请确保所有数据库都放置在本地 SSD 上。

If you have some databases on an SSD in the `<SSD_DB_DIR>` directory and some
databases on a slower disk in the `<DB_DIR>` directory, you can mount both
directories and specify `db_dir` multiple times. This will enable the fast
access to databases with a fallback to the larger, slower disk:
如果您将部分数据库放在 SSD 的 <SSD_DB_DIR> 目录中，而将其他数据库放在较慢磁盘的 <DB_DIR> 目录中，您可以同时挂载这两个目录并多次指定 db_dir。这将实现对数据库的快速访问，同时还能回退到更大但更慢的磁盘：

```sh
docker run -it \
    --volume $HOME/af_input:/root/af_input \
    --volume $HOME/af_output:/root/af_output \
    --volume <MODEL_PARAMETERS_DIR>:/root/models \
    --volume <SSD_DB_DIR>:/root/public_databases \
    --volume <DB_DIR>:/root/public_databases_fallback \
    --gpus all \
    alphafold3 \
    python run_alphafold.py \
    --json_path=/root/af_input/fold_input.json \
    --model_dir=/root/models \
    --db_dir=/root/public_databases \
    --db_dir=/root/public_databases_fallback \
    --output_dir=/root/af_output
```

If you get an error like the following, make sure the models and data are in the
paths (flags named `--volume` above) in the correct locations.
如果您遇到以下错误，请确保模型和数据位于正确位置的路径（上面名为`--volume`的标志）中。

```
docker: Error response from daemon: error while creating mount source path '/srv/alphafold3_data/models': mkdir /srv/alphafold3_data/models: permission denied.
```

`run_alphafold.py` supports many flags for controlling performance, running on
multiple input files, specifying external binary paths, and more. See

```sh
docker run alphafold3 python run_alphafold.py --help
```

for more information.

## Running Using Singularity Instead of Docker 使用Singularity而不是Docker运行

You may prefer to run AlphaFold 3 within Singularity. You'll still need to
*build* the Singularity image from the Docker container. Afterwards, you will
not have to depend on Docker (at structure prediction time).
您可能更喜欢在Singularity中运行AlphaFold 3。你仍然需要从Docker容器中构建Singularity镜像。之后，您将不必依赖Docker（在结构预测时）。

### Install Singularity 安装Singularity

Official Singularity instructions are
[here](https://docs.sylabs.io/guides/3.3/user-guide/installation.html). The
commands we ran are:

```sh
wget https://github.com/sylabs/singularity/releases/download/v4.2.1/singularity-ce_4.2.1-jammy_amd64.deb
sudo dpkg --install singularity-ce_4.2.1-jammy_amd64.deb
sudo apt-get install -f
```

### Build the Singularity Container From the Docker Image 从Docker镜像构建Singularity容器

After building the *Docker* container above with `docker build -t`, start a
local Docker registry and upload your image `alphafold3` to it. Singularity's
instructions are [here](https://github.com/apptainer/singularity/issues/1537).
The commands we ran are:
在使用`Docker build-t`构建了上面的*Docker*容器后，启动一个本地Docker注册表，并将您的镜像`alphafold3`上传到其中。Singularity的说明如下[此处](https://github.com/apptainer/singularity/issues/1537).
我们运行的命令是：

```sh
docker run -d -p 5000:5000 --restart=always --name registry registry:2
docker tag alphafold3 localhost:5000/alphafold3
docker push localhost:5000/alphafold3
```

Then build the Singularity container:
然后构建Singularity容器：

```sh
SINGULARITY_NOHTTPS=1 singularity build alphafold3.sif docker://localhost:5000/alphafold3:latest
```

You can confirm your build by starting a shell and inspecting the environment.
For example, you may want to ensure the Singularity image can access your GPU.
You may want to restart your computer if you have issues with this.
您可以通过启动shell并检查环境来确认您的构建。例如，您可能希望确保Singularity映像可以访问您的GPU。如果您遇到此问题，可能需要重新启动计算机。

```sh
singularity exec --nv alphafold3.sif sh -c 'nvidia-smi'
```

You can now run AlphaFold 3!

```sh
singularity exec --nv alphafold3.sif <<args>>
```

For example:

```sh
singularity exec \
     --nv \
     --bind $HOME/af_input:/root/af_input \
     --bind $HOME/af_output:/root/af_output \
     --bind <MODEL_PARAMETERS_DIR>:/root/models \
     --bind <DB_DIR>:/root/public_databases \
     alphafold3.sif \
     python run_alphafold.py \
     --json_path=/root/af_input/fold_input.json \
     --model_dir=/root/models \
     --db_dir=/root/public_databases \
     --output_dir=/root/af_output
```

Or with some databases on SSD in location `<SSD_DB_DIR>`:

```sh
singularity exec \
     --nv \
     --bind $HOME/af_input:/root/af_input \
     --bind $HOME/af_output:/root/af_output \
     --bind <MODEL_PARAMETERS_DIR>:/root/models \
     --bind <SSD_DB_DIR>:/root/public_databases \
     --bind <DB_DIR>:/root/public_databases_fallback \
     alphafold3.sif \
     python run_alphafold.py \
     --json_path=/root/af_input/fold_input.json \
     --model_dir=/root/models \
     --db_dir=/root/public_databases \
     --db_dir=/root/public_databases_fallback \
     --output_dir=/root/af_output
```
