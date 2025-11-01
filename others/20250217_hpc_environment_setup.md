## HPC Cluster Environment Setup Guide

This guide provides step-by-step instructions for setting up the computing environment on our HPC cluster. It includes removing old installations, installing `micromamba`, setting up the Script of Scripts (SoS) computing environment, and managing data storage efficiently.



# Removing Old Installations (Skip if You Are a New User)

Before installing `micromamba`, ensure that any previous installations are removed or backed up. Check for existing installations in the following directories:

```shell
ls ~/bin/micromamba ~/bin/pixi ~/.local/micromamba ~/.local/pixi ~/micromamba ~/pixi ~/.micromamba ~/.pixi
```

To back up these directories before removal, you can rename them:

```shell
mv ~/micromamba ~/micromamba_backup
```

Once the new setup is confirmed to be working correctly, you may delete the backup directories.



# Configure the network proxy

To configure the network proxy, add the following commands to your `~/.bashrc` and then run the `source` command. Begin by opening `~/.bashrc` in a text editor and appending the commands:

```
export http_proxy=http://menloproxy.cumc.columbia.edu:8080
export https_proxy=http://menloproxy.cumc.columbia.edu:8080
```

and type `source ~/.bashrc` to load the changes.



# Install `micromamba`

We recommend using `micromamba` over `miniconda` or `anaconda` because:

- It does not require a `base` environment.
- It does not come with a default Python version, allowing greater flexibility.
- It is implemented in C++ and runs faster than `conda`.

## Installation Steps

Follow the official installation instructions [here](https://mamba.readthedocs.io/en/latest/micromamba-installation.html). Alternatively, use `wget` to install `micromamba` as shown below:

```shell
cd ~
wget -qO- https://micromamba.snakepit.net/api/micromamba/linux-64/latest | tar -xvj bin/micromamba
~/bin/micromamba shell init -s bash ~/micromamba
```

where you manually specific the OS, in this case `linux-64`. 

After installation, update your shell configuration:

```shell
source ~/.bashrc
```

## Verifying Installation

To confirm that `micromamba` is installed correctly, run:

```shell
micromamba -h
```

This should display the help message.

## Configuring Default Channels

For convenience, add commonly used package channels:

```shell
micromamba config prepend channels nodefaults
micromamba config prepend channels bioconda
micromamba config prepend channels conda-forge
```

# Setting Up the SoS Computing Environment (pieces-rabbit)

We use the Script of Scripts (SoS) suite along with **Python and R** for computational workflows. The recommended version can be installed using the following configuration file: [`pisces-rabbit.yml`](https://github.com/gaow/misc/blob/master/docker/pisces-rabbit.yml).

## Installation

Run the following commands to install the SoS environment:

```shell
wget https://raw.githubusercontent.com/gaow/misc/master/docker/pisces-rabbit.yml 
micromamba env create -y -f pisces-rabbit.yml
```

- (optional): The environment is named based on the Zodiac sign of the month and the corresponding year. For example, `pisces-rabbit` corresponds to a stable setup tested by our lab members as of February 2023 (Pisces, Rabbit year). This guide will be updated periodically with the latest stable versions.

## Activating the Environment

To load this environment automatically when opening a new shell session, add the following line to your `~/.bashrc` file:

```shell
micromamba activate pisces-rabbit 
```

Then apply the changes:

```shell
source ~/.bashrc
```

If you prefer to activate it manually, type this directly in the command line:

```shell
micromamba activate pisces-rabbit
```

## Notes for Neurology HPC Users

### Job Submission Considerations

On the Neurology HPC cluster, computing nodes do **not** automatically source your `~/.bashrc` settings. To ensure the environment is correctly set up in job scripts, include the following lines:

```shell
export PATH=$HOME/.local/bin:$PATH
export MAMBA_ROOT_PREFIX=$HOME/micromamba
eval "$(micromamba shell hook --shell bash)"
micromamba activate pisces-rabbit
```

To simplify this process, save these lines in a script called `~/mamba_activate.sh`, and include this command at the beginning of your job submission script:

```shell
source ~/mamba_activate.sh
```

### Installing the `sos-r` Plugin

By default, the SoS environment does not include `sos-r`. Neurology HPC users who require R functionality in SoS notebooks should install it manually:

```shell
micromamba activate pisces-rabbit
micromamba install sos-r -y
```

# Data Storage on the HPC Cluster

## Personal Data Storage

To efficiently manage storage, users should **not** store all data in their home directory (`/home/YOUR_UNI`). Instead, create a dedicated folder in the shared storage system:

```shell
# Create a personal folder in the shared storage
cd /mnt/vast/hpc/csg/
mkdir $YOUR_UNI
```

This ensures proper organization and allows better space management.

## Shared Data Storage

Public datasets that are accessible to all users are stored in:

```shell
/mnt/vast/hpc/csg/data_public/
```

These datasets are typically **read-only** for most users.

## Best Practices for Data Management

- **Avoid redundant copies**: Store data in shared locations instead of making multiple copies.
- **Follow permissions and access rules**: Public data can usually be read but not modified.
- **Organize your workspace**: Keep job scripts, analysis notebooks, and GitHub repositories structured within `/mnt/vast/hpc/csg/$<YOUR_UNI>/`.
- **Be mindful when working with shared data**: Ensure that you do not accidentally modify or overwrite files owned by others.

# Slurm job management

Slurm (Simple Linux Utility for Resource Management) is a widely used open-source workload manager designed for high-performance computing (HPC) clusters. It efficiently allocates resources, schedules jobs, and manages parallel computing workloads. Slurm enables users to submit, monitor, and manage computational jobs while optimizing cluster utilization.

Our HPC cluster uses Slurm for job scheduling. Below, we provide a basic guide to submitting and managing jobs.

## Submitting a Test Job

To get started, you can use the following **SBATCH** script located at `/home/rd2972/test.sbatch`, which serves as a minimal example of a Slurm job submission script:

```shell
#!/bin/bash
#SBATCH --job-name=test       # Job name
#SBATCH --mem=1G              # Memory allocation
#SBATCH --time=10:00:00      # Maximum runtime
#SBATCH --output=./test_%j.out  # Standard output log
#SBATCH --error=./test_%j.err   # Standard error log
#SBATCH -p CSG                # Partition name

which python
which R

echo "hello world"
```

This script requests **1GB of memory** and runs for a maximum of **10 hours** on the **CSG** partition. It prints the paths of the currently loaded Python and R installations and outputs "hello world" to confirm execution.

## Running the Test Job

To submit the job, first copy the script to your working directory and execute the following command:

```shell
cp /home/rd2972/test.sbatch ./
sbatch ./test.sbatch
```

Upon submission, Slurm assigns a job **ID** and creates two output files in your directory:

- `.out` file: Stores standard output (stdout) messages
- `.err` file: Stores standard error (stderr) messages

These files help debug and verify that your job runs correctly.

## Managing Jobs with Slurm

Here are some common Slurm commands for job management:

```shell
# Submit a job
sbatch test.sbatch

# Cancel a job (replace 1719337 with your job ID)
scancel 1719337

# Check the status of your jobs
squeue --me

# Start an interactive session with 20GB of memory
srun --pty --mem=20G bash
```

**Best Practices**

- **Request only the necessary resources** (memory, time, CPUs) to avoid inefficient resource allocation.
- **Start with small memory requests** when testing to minimize queue wait times and prevent excessive cluster load.
- **Check job logs (`.out` and `.err` files)** to debug any issues before resubmitting jobs.

# Jupyter notebook

The script to submit a jupyter notebook job reads like (make sure you have created `~/mamba_activate.sh` as above):

```shell
#!/bin/bash
#SBATCH --mem=50G
#SBATCH --time=360:00:00
#SBATCH --job-name=jupyter_notebook_SLURM
#SBATCH --output=z_jupyter_notebook_%j.out
#SBATCH -p CSG

# get tunneling info
XDG_RUNTIME_DIR=""
port=$(shuf -i8000-9999 -n1)
node=$(hostname -s)
user=$(whoami)
cluster=csglogin.neuro.columbia.edu

# print tunneling instructions jupyter-log
echo -e "

MacOS or linux terminal command to create your ssh tunnel
ssh -N -L ${port}:${node}:${port} ${user}@${cluster}

Windows MobaXterm info
Forwarded port:same as remote port
Remote server: ${node}
Remote port: ${port}
SSH server: ${cluster}
SSH login: $user
SSH port: 22

Use a Browser on your local machine to go to:
https://localhost:${port}  (prefix w/ http:// instead if the browser complains that secured connection cannot be established)
" > z_jupyter_notebook_$SLURM_JOBID.login_info

# Start jupyter
cd
source ~/mamba_activate.sh
jupyter-lab --no-browser --port=${port} --ip=${node}
```

save this script to `~/jupyter_job_slurm.sbatch` and you can submit this by running `sbatch ~/jupyter_job_slurm.sbatch`



