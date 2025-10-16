# TUNI Narvi Cluster

*This amazing guide was originally posed in [wiki.eduuni.fi](https://wiki.eduuni.fi/) and composed by Heikki Huttunen. We obtained his permission to use it here. It was modernized and expanded since then.*

This document describes how to use the `TUNI TCSC Narvi` computing cluster.

!!! question "What is Narvi?"
    * Narvi is the SLURM cluster that substituted the old merope cluster in 2017.
    * The cluster is used by researchers, faculty members, and students at Tampere University.
    * There are 140 CPU-only nodes with 3000+ CPU cores
    * Also Narvi has 22 nodes with different GPU nodes with 4 GPUs in each. Specifically, there are
        * 6 nodes with Tesla V100 32 GB and 4 with Tesla V100 16 GB
        * 2 nodes with Tesla P100 16 GB and 6 with Tesla P100 12 GB
        * 4 nodes with Tesla K80 12 GB

## How to Get an Account?

1. Go to [id.tuni.fi](https://id.tuni.fi/)
2. Choose `Identity management` → `My user rights` → `Apply for a new user right`
3. Choose the correct contract (studen/staff).
    - If you fill the application as a student also tell the Course name and the responsible teacher
4. Search for `Linux Servers (LINUX-SERVERS) TCSC HPC Cluster`
5. In the form, enter application details: `I need Narvi account. My supervisor is X.`.
6. Your supervisor will receive an acceptance link and you will be granted a new account.
7. In a few days, you will receive an email from TCSC telling you to send an ssh public key to them. Create a key pair (see below or just google for it) and send the *public* one to them.
8. Soon you will be able to login to the front end `narvi.tut.fi` using `ssh`.

??? question "How to generate an `ssh` key-pair 🔐?"
    Here is how to do it on Linux and Mac systems. The instructions for Windows can be easily found on google.
    ```bash
    ssh-keygen -f ~/.ssh/narvi_key
    ```
    The command will ask for a new password which will be asked when this key is used. After, the script will save two keys (`narvi_key` and `narvi_key.pub`) in `~/.ssh/` folder. `*.pub` is the public key.

    The first time you will use this key with `ssh` it may complain about permissions (`Permissions are too open.`). If so, you will need to change the permissions of the private key
    ```bash
    chmod 600 ~/.ssh/narvi_key
    ```

??? question "I don't see any GPU partitions"
    Please write an e-mail to the admin (`tcsc.tau@tuni.fi`) asking to add you to the GPU group. By default, you will only have access to CPU-only nodes.

## How to Check the Queue

To see the status of the queue, type
```bash
squeue
# for a specific partitions (e.g. `normal` or `gpu`).
squeue -p gpu
# for a specific user
squeue -u <user>
```

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

To learn more `sbatch` hacks, a reader is also referred to [this nice tutorial](https://narvi-docs.readthedocs.io/narvi/tut/gpu.html).

## How to Cancel My Job
To cancel a specific job you are running, use
```bash
scancel <JobID>
```

## How to Transfer Data?

The simplest way is to use `scp` command
```bash
scp -i ~/.ssh/narvi_key -r ./folder user@narvi.tut.fi:/narvi/path/
```
where `-r` means to copy the folder with all files in it.

However, once the internet connection is interrupted you will need to start all over again. To have an opportunity to resume the data transfer try `rsync` instead
```bash
rsync -ahP -e "ssh -i ~/.ssh/narvi_key" ./folder user@narvi.tut.fi:/narvi/path/
```
where `-ah` means to preserve permissions symlinks, etc as in the original folder and `h` makes the progress "human-readable", and `P` allows to continue data transfer (sends missing files on the target path 🤓).

!!! warning "Trailing `/` in `rsync` makes the difference"
    - `rsync /dir1/dir2/ /home/dir3` - copies the contents of `/dir1/dir2` but not the `dir2` folder itself.
    - `rsync /dir1/dir2 /home/dir3` – copies the folder `dir2` along with all its contents.

If you would like to see the files from a remote machine you may mount the folder locally. On Ubuntu/Debian install `sshfs` and run this
```bash
mkdir narvi_folder
sshfs -o IdentityFile=~/.ssh/narvi_key user@narvi.tut.fi:/narvi/folder/ ./narvi_folder
```
the content of the `/narvi/folder` will be shown in `./narvi_folder`. Mind that the changes in either folder will be reflected in another one.

To unmount the folder use
```bash
umount ./narvi_folder
```

## How Do I Install My Software
Before you do so, check if the software you would like to install is already installed by the admin (e.g. matlab, cuda, and gcc). These are set up using `module` functionality. You can load a module by specifying `module load <mod>` inside of your script. To see all available modules run `module avail`.

If you are not satisfied with the selection you can install your own. Here we will focus on `Python` packages and virtual environment manager `conda` which is already installed on Narvi (try: `which conda`).

!!! tip "`conda` Has Many Linux Tools"
    Besides a ton of `Python` packages, `conda` has surprisingly many common Linux tools, e.g. `tmux`, `htop`, `ffmpeg`, `vim`, and more. This is especially useful if you would like to install them but do not have `sudo` rights.

### Creating a `conda` Environment

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
