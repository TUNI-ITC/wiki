# TUNI TCSC Cluster

*This guide is based on similar one made for the previous TUNI cluster Narvi and can be found from X Files of this Wiki ([TUNI Narvi Cluster](../X-Outdated/tuni-narvi-cluster.md))*

This document describes how to use the `TUNI TCSC` computing cluster.

!!! question "What is TCSC?"
    * Narvi is the SLURM cluster that substituted the old Narvi cluster in 2025 (that replaced even older Merope cluster in 2017).
    * The cluster is used by researchers, faculty members, and students at Tampere University.
    * There are powerful CPU-only (e.g. Xeon) and GPU nodes (e.g. 4xV100 and 8xH200) - details about available resources can be found from the [TCSC Web page](https://www.tuni.fi/en/research/tampere-center-scientific-computing-tcsc).

## How to Get an Account?

The user rights are granted per research team and maintained by the team leader (PI). Details about obtaining the rights is found from [TUNI Intranet](https://intra.tuni.fi/en/it-services/it-research/storing-and-processing-research-data-0/tcsc-high-performance-computing-cluster) and it can take several days.

Once you have the accounts ready, you can access the computing resources through a dedicated Web interface or old-fashioned using terminal with ssh. Note that outside the campus network VPN connectin can be needed.

## How to Check the Queue

To see the status of the queue, type
```bash
squeue
# for a specific partitions (e.g. `test`).
squeue -p test
# for a specific user
squeue -u <user>
```

Available job queues are listed in the internal [TCSC wiki](https://tuni.atlassian.net/wiki/spaces/TCSC/pages/332562812/Available+batch+job+partitions) (access is limited).

## How to Run a Job


*Remember: do not use the login node for computation – it is slow and will degrade the performance of the login node for other users!*

There are two common ways to run a job at a `slurm` cluster:

- `srun`
- `sbatch`

The main difference is that `srun` is interactive which means the terminal will be attached to the current session. The experience is just like with any other command in your terminal. Note, that when the queue is full you will have to wait until you get resources.

If you use `sbatch`, you submit your job to the slurm queue and get your terminal back; you can disconnect, kill your terminal, etc. with no consequence. In the case of `srun`, killing the terminal would kill the job. Hence, `sbatch` is recommended.

Here is the example `srun` command which will ask the cluster to start an interactive shell with 1 GPU (`--gres`) and 10 CPUs (`--cpus-per-task`), 10 GB of RAM (`--mem-per-cpu`) that will be available to you for 30 minutes (`--time`):
```
srun \
    --pty \
    --job-name pepe_run \
    --partition gpu \
    --gres gpu:1 \
    --mem-per-cpu 1G \
    --ntasks 1 \
    --cpus-per-task 10 \
    --time 00:30:00 \
    /bin/bash -i
```

and this is an example `sbatch` command which will ask the cluster to run `my_script.sh` with 1 GPU and 10 CPUs, 10 GB of RAM that will run for at most 30 minutes (if the script has finished execution the job will be ended), the output and error logs will be saved to `log_JOBID.txt` (`--output`, `--error`):
```
sbatch \
    --job-name pepe_run \
    --partition gpu \
    --gres gpu:1 \
    --mem-per-cpu 1g \
    --ntasks 1 \
    --cpus-per-task 10 \
    --time 00:30:00 \
    --output log_%j.txt \
    --error log_%j.txt \
    my_script.sh
```
you may also use `--constraint='kepler|pascal|volta'` in order to select a specific gpu architecture.

Instead of specifying the resources and other information as command-line arguments, you may find it useful to list them inside of `my_script.sh` and then just use `sbatch my_script.sh`:
```
#!/bin/bash
#SBATCH --job-name=pepe_run
#SBATCH --gres=gpu:1
#SBATCH --time=00:30:00
# and so on. To comment SBATCH entry use `##SBATCH --arg ...`
# here starts your script
```

To learn more `sbatch` hacks, a reader is also referred to [this nice tutorial](https://www.arch.jhu.edu/short-tutorial-how-to-create-a-slurm-script/).

## How to Cancel My Job
To cancel a specific job you are running, use
```bash
scancel <JobID>
```

## How to Transfer Data?

The simplest way is to use `scp` command
```bash
scp -r ./folder user@xxxx.tuni.fi:/my/path/
```
where `-r` means to copy the folder with all files in it.

However, once the internet connection is interrupted you will need to start all over again. To have an opportunity to resume the data transfer try `rsync` instead
```bash
rsync -ahP ./folder user@xxxx.tuni.fi:/my/path/
```
where `-ah` means to preserve permissions symlinks, etc as in the original folder and `h` makes the progress "human-readable", and `P` allows to continue data transfer (sends missing files on the target path 🤓).

!!! warning "Trailing `/` in `rsync` makes the difference"
    - `rsync /dir1/dir2/ /home/dir3` - copies the contents of `/dir1/dir2` but not the `dir2` folder itself.
    - `rsync /dir1/dir2 /home/dir3` – copies the folder `dir2` along with all its contents.

If you would like to see the files from a remote machine you may mount the folder locally. On Ubuntu/Debian install `sshfs` and run this
```bash
mkdir narvi_folder
sshfs user@xxx.tuni.fi:/my/folder/ ./my_folder
```
the content of the `/my/folder` will be shown in `./my_folder`. Mind that the changes in either folder will be reflected in another one.

To unmount the folder use
```bash
umount ./my_folder
```

## How Do I Install My Software

### Slurm modules (best option)
Before you install own software, check if the software you would like to install is already installed by the admin (e.g. matlab, cuda, and gcc). These are set up using `module` functionality. You can load a module by specifying `module load <mod>` inside of your script. To see all available modules run `module avail`.

Examples are given in the [TCSC internal Wiki](https://tuni.atlassian.net/wiki/spaces/TCSC/pages/332562768/Module+environment), and a good tutorial is provided by CSC in their [Slurm Modules tutorial slide set](https://a3s.fi/CSC_training/04_modules.html#/module-systems-in-supercomputers).

If you are not satisfied with the selection you can install your own. Below are several options how to do that.


### Using Conda (avoid if you can)
Conda allows you to

 - Fully control what software packages are installed
 - Have own *environments* for different projects
 - Copy environments from your computer to a TCSC computing node

However, using conda **is not recommended** since

 - <font color="red">Problems to use together with Slurm modules</font>
 - <font color="red">sbatch startup can be slow</font> - read more from [CSC best practices for Conda](https://docs.csc.fi/support/tutorials/conda/)

!!! tip "`conda` Has Many Linux Tools"
    Besides a ton of `Python` packages, `conda` has surprisingly many common Linux tools, e.g. `tmux`, `htop`, `ffmpeg`, `vim`, and more. This is especially useful if you would like to install them but do not have `sudo` rights.

#### Creating a `conda` Environment

Minimal Conda is available in the *miniforge* package, log in the tcsc computer and load the module
```bash
module load miniforge3/24.9.0
```

Let's start by creating an empty conda environment
```bash
conda create --name my_env
```

Activate it (meaning that all binaries installed in this environment will be used instead of the system-wise packages)
```bash
conda activate my_env
# if it didn't work try `source activate my_env`
```

Afterward, you can install `conda` packages
```bash
conda install python pip matplotlib scikit-learn
```

If default `conda` channels don't have some package you search for other `conda` channels:
```bash
conda install dlib --channel=menpo
```

If your favorite package is not available anywhere in `conda` _OR you would like to install `OpenCV`_, try to install it via `pip`:
```bash
# check if you are using the `pip` from your `conda` env
which pip
pip install opencv-python
```

??? question "`conda` vs `pip` inside of `conda` env?"
    According to [official anaconda documentation](https://www.anaconda.com/blog/using-pip-in-a-conda-environment), you should install as many requirements as possible with `conda`, then use `pip`.
    Another problem with `pip` packages inside of `conda` is associated with poor dependence handling and just bad experience when trying to replicate the same environment on another machine.

Alternatively, if you have an existing Conda environment for your code, it can be exported
```bash
conda activate my_env
conda env export > my_env.yml
```
The yml file is transferred to the TCSC computer where it can be used to install the same environment:
```bash
module load miniforge3/24.9.0
conda env create -f my_env.yml
# This can take long
conda activate my_env
```

Several guides could be useful:

 - Problems to run Conda with sbatch - [Activating Conda Environments from Scripts: A Guide for Data Scientists](https://saturncloud.io/blog/activating-conda-environments-from-scripts-a-guide-for-data-scientists/)
 - [Conda best practices for CSC supercomputers](https://docs.csc.fi/support/tutorials/conda/)
 

### Using Conda and existing modules (Better than only Conda)
<font color="red">Someone must write</font>

### Using Python virtual environment
<font color="red">Someone must write</font>

### Using containers
<font color="red">Someone must write</font>
