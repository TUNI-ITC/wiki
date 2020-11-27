# TUNI Narvi Cluster

*This amazing guide was originally posed in [wiki.eduuni.fi](https://wiki.eduuni.fi/) and composed by Heikki Huttunen. We obtained his permission to repost it here.*

This document describes how to use the TUNI TCSC Narvi computing cluster, and describes in particular how to use the GPU cores using the Keras package. The package implements many recent deep learning methodologies in a highly configurable manner and is implemented in `Python`.

!!! info "What is Narvi?"
    * Narvi is the SLURM cluster that substituted the old merope cluster in 2017.
    * The cluster is used by researchers, faculty members, and students at Tampere University.
    * There are 140 CPU-only nodes with 3000+ CPU cores
    * Also Narvi has 22 nodes with different GPU nodes with 4 GPUs in each. Specifically, there are
        * 6 nodes with Tesla V100 32 GB and 4 with Tesla V100 16 GB
        * 2 nodes with Tesla P100 16 GB and 6 with Tesla P100 12 GB
        * 4 nodes with Tesla K80 12 GB

## How to Get an Account?

1. Go to [omatunnus](http://www.tut.fi/omatunnus/)
2. Open "Services" and make sure the "Services" tab is selected.
3. Click "system access"
4. Click "IT". Click "Servers". Click "TCSC". Scroll down & click "Select".
5. In the form, enter additional information: "I need Narvi account. My supervisor is N.N.".
6. Your supervisor will receive an acceptance link and you will be granted a new account. Soon you can log in to Narvi front end: `ssh narvi.tut.fi`
7. In a few days, you will receive an email from TCSC telling you to send an ssh public key to them. Create a keypair (google "ssh keypair generation" for instructions) and send the **public** one to them.
8. Soon you will be able to login to the SLURM front end `narvi.tut.fi` using `ssh`.

## Commands on the Front-end

To list all cluster partitions available for your account type
```bash
sinfo
# or
# sinfo -o "%20N  %10c  %10m  %25f  %10G "
```

??? info "I don't see any GPU partitions"
    E-mail the current admin of Narvi asking to add you to the GPU group. By default, you will only have access to CPU-only nodes.

To see the status of the queue type
```bash
squeue
```
Use `-u` argument to filter the queue for a specific user (`sbatch -u <user>`) or `-p` to filter for a partition (e.g. `normal` or `gpu`).

<!-- add a link to the instructions within this doc -->
(The instructions on how to run code are provided later in this document.) To cancel a specific job you are running, use
```bash
scancel <JobID>
```


## How to Run a Job

There are two common ways to run a job at a `slurm` cluster: using `sbatch` and `srun` commands.

The main difference is that `srun` is interactive which means the terminal will be attached to the current session. The experience is just like with any other command in your terminal. Note, that when the queue is full you will have to wait until you get resources.

If you use `sbatch`, you submit your job to the slurm queue and get your terminal back; you can disconnect, kill your terminal, etc. with no consequence. In the case of `srun`, killing the terminal would kill the job. Hence, `sbatch` is recommended.

Here is the example command which will ask the cluster to run `my_script.sh` with 1 GPU and 10 CPUs, 10 GB of RAM that will run for 30 minutes:
```
sbatch --job-name pepe_run \
    --partition=gpu \
    --gres=gpu:1 \
    --mem-per-cpu=1G \
    --ntasks=1 \
    --cpus-per-task=10 \
    --time=00:30:00 \
    my_script.sh
```
you may also use `--constraint='kepler|pascal|volta'` in order to select a specific GPU architecture.

Instead of specifying the resources and other information as command-line arguments, you may find it useful to list them inside of `my_script.sh`:
```bash
#!/bin/bash
#SBATCH --job-name=pepe_run
#SBATCH --gres=gpu:1
#SBATCH --time=00:30:00
# and so on. To comment SBATCH entry use `##SBATCH --arg ...`
# here starts your script
```

To learn more `sbatch` hacks, a reader is also referred to [this nice tutorial](https://narvi-docs.readthedocs.io/narvi/tut/gpu.html).

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
