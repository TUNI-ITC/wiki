# An automation script to run your job to Puhti or Mahti

## Table of Contents

1. [Motivation](#motivation)
2. [Dislaimer](#disclaimer)
3. [Introduction](#introduction)
    1. [What is the automation script?](#what-is-the-automation-script)
    2. [Why use the automation script?](#why-use-the-automation-script)
    3. [How does the automation script work?](#how-does-the-automation-script-work)
    4. [What are the requirements for using the automation script?](#what-are-the-requirements-for-using-the-automation-script)
    5. [What are the limitations of the automation script?](#what-are-the-limitations-of-the-automation-script)
4. [How to use it](#how-to-use-it)
    1. [How to start a new job](#how-to-start-a-new-job)
    2. [How to check the status of a job](#chow-to-heck-the-status-of-a-job)
    3. [How to stop a running job](#how-to-stop-a-running-job)
5. [How to use this script for PyTorch model training](#how-to-use-this-script-for-pytorch-model-training)
    1. [How to modify the PyTorch-based model training script](#how-to-modify-the-pytorch-based-model-training-script)
    2. [Some notifications for the command you want to run](#some-notifications-for-the-command-you-want-to-run)
    3. [Distributed Data Parallel (DDP) with PyTorch](#distributed-data-parallel-ddp-with-pytorch)
6. [Some errors and solutions](#some-errors-and-solutions)
7. [Contact](#contact)

## Motivation

If you want to train some huge models, you might want to use the supercomputers like Puhti or Mahti. However, due to the
limitations from SLURM clusters, the process of submitting a job to the supercomputers is not straightforward. You need
to write a SLURM script, submit the job, and check the status of the job. This process is time-consuming and needs you
to submit the job manually, which will lead to lots of time wasted if you couldn't apply for new resources after the
current work is done. Therefore, we provide an automation script to help you submit your job to Puhti or Mahti. You
don't need to worry that your job stops running during the night or your holiday anymore. Just enjoy your time and let
the automation script do the rest for you.

## Disclaimer

- Important! Please follow the rules of CSC when using this script! If you have any questions about the rules, please check the
  information on [Docs CSC](https://docs.csc.fi/).
- This script is binary encrypted, and the source code is not available. Please use it following the instructions and do
  not modify it.
- This script is only available for Puhti or Mahti.
- This script is only available for non-interactive jobs. If you want to run an interactive job, you need to submit the
  job manually.
- This script is only available for 4 jobs at the same time. If you want to run more jobs, you need to stop some of them
  first.
- If you got any problems or bugs when using this script, please [contact me](#contact).

## Introduction

### What is the automation script?

This is a encrypted shell script that can help you submit your job to Puhti or Mahti.

### Why use the automation script?

It will automatically apply for some resources and submit your job to the supercomputers. After the current job gets
close to the time limit, the script will automatically apply for new resources and submit the job again. What's more,
the old resources will be released immediately when the new resources are allocated. You don't need to worry that your
job stops running due to the time limit of the server anymore.

### How does the automation script work?

This automation script will run as the 5 steps below:

- Step 1: Apply for the required resources on SLURM clusters
- Step 2: Start to timing as soon as getting the resources
- Step 3: Submit a new request for resources when the remaining time is 1/3 of the whole requested time
- Step 4: Stop the current job and release the resources as soon as the new resources are allocated
- Step 5: Repeat the steps above

### What are the requirements for using the automation script?

- You need to have a user account available on Puhti or Mahti
- You need to put this automation script into your user directory on Puhti or Mahti
- If this script couldn't be executed, you need to give the execution permission to it by
  running `chmod +x auto_gpu`

### What are the limitations of the automation script?

- This script is only available on Puhti or Mahti
- This script limits the maximum of jobs you can run at the same time to 4 in order to avoid the congestion and the
  waste of resources
- This script is not available for psuedo-interactive jobs, which will lead to lots of resources wasted. If you want to
  run a psuedo-interactive job, you need to submit the job manually
- If you want to train a model with PyTorch for a long while, which would be split into several jobs due to the time
  limit of the server, you need to modify the PyTorch-based model training script. (details
  in [How to modify the PyTorch-based model training script](#how-to-modify-the-pytorch-based-model-training-script))
- This script can not stop automatically. You need to stop it manually when you want to stop the automation script. (
  details in [How to stop a running job](#how-to-stop-a-running-job))
  And please remember to stop the script after your work is done to avoid the waste of resources.

## How to use it

Please firstly download the script from the following link: [auto_gpu](assets/auto_gpu). Then you can put it into your
user folder. And you can run the script by the following command:

```shell
./auto_gpu
```

If the permission is denied, you can give the execution permission to it by running the following command:

```shell
chmod +x auto_gpu
```

Then it can work properly.

### How to start a new job

This script provides a lot of options for you to customize your job. If you want to check the help information, you can
choose one of the following commands:

```shell
./auto_gpu -h
./auto_gpu --help
```

And the information will be shown as below:

```shell
Options:
  -h, --help                            Display help.
                                        
  -j <job_name>, --job_name=<job_name>  Set the job name you want to use. No default value. Please set different names for different jobs! This is an identifier for the job.
                                        
  -a <account>, --account=<account>     Set the account you want to use. No default value.
                                        
  -t <time>, --want_time=<time>         Set the time you want to use the GPU (!!!the maximum of one round, not the whole time!!!), and the format must be 'D-HH:MM:SS' (D: days, HH: hours, MM: minutes, SS: seconds). The default is 0-00:15:00.
                                        
  -s, --small                           Use gpusmall on mahti.
  -m, --medium                          Use gpumedium on mahti.
  -l, --large                           Use gpu on puhti.
                                        
  -n <num>, --num_nodes=<num>           Set the number of nodes you want to use. The default is 1.
                                        
  -g <num>, --num_gpus_per_node=<num>   Set the number of GPUs per node you want to use. The default is 1.
                                        
  --nvme=<num>, --nvme_per_node=<num>   Set the local storage per node (GB) you want to use. No default value.
                                        
  --num_tasks_per_node=<num>            Set the number of tasks per node you want to use. The default is 1.
                                        
  -c <num>, --num_cpus_per_task=<num>   Set the number of CPUs per task you want to use. The default is 4.
                                        
  --mem_per_cpu=<num>                   Set the memory per CPU you want to use. The default is 8000.
                                        
  --cmd=<command>, --command=<command>  Set the command you want to run, it must be double quote. No default value.
```

Now, if we want to apply for 1 node with 1 GPU (gpusmall) on Mahti for 1 hour to run <your_command>, we can run one of
the following commands:

```shell
./auto_gpu -t 0-01:00:00 -s -j <your_job_name> -a <your_csc_group_name> --cmd="<your_command>"
./auto_gpu --want_time=0-01:00:00 --small --job_name=<your_job_name> --account=<your_csc_group_name> --command="<your_command>"
```

If you want to apply for different GPU type, more time, nodes, GPUs, CPUs, or memory, you can customize the options as
you want.

### How to check the status of a job

This script will automatically output the job ID and the node name immediately after getting the resources. You can
check it directly from the terminal. If you want to check the status of the job (like partition, name, state, time,
etc.), you can run the following command:

```shell
squeue -u <your_username>
```

### How to stop a running job

If you want to stop a running job, You can do the following steps:

- Step 1. Press `Ctrl + C` to stop the script
- Step 2. Run the following command to check which job you want to stop, and copy the job ID

```shell
squeue -u <your_username>
```

- Step 3. Run the following command to stop the job

```shell
scancel <job_id>
```

## How to use this script for PyTorch model training

### How to modify the PyTorch-based model training script

If you want to train a model with PyTorch for a long while, which might be split into several rounds due to the time
limit of the server. In this case, the model will be refreshed every time you submit a new job. Therefore, you need to
save the parameters of the model every epoch, and load the latest parameters when you start a new job. Here is an
example of how to modify the PyTorch-based model training script:

```python
# save the model after each epoch
import os

torch.save(model.state_dict(), '/<your_model_path>/<your_model_name>_{:d}.pth'.format(epoch_idx))

# remove the old model automatically (If you want to check some old models, you can remove this line)
# But please remember to clean the old models in time to avoid the waste of resources
os.remove('/<your_model_path>/<your_model_name>_{:d}.pth'.format(epoch_idx - 1))

# load the latest model
epoch_list = [int(model_name.split('_')[-1].split('.')[0]) for model_name in os.listdir('/<your_model_path>/') if
              model_name.endswith('.pth')]
max_epoch = max(epoch_list)
model.load_state_dict(
    torch.load('/<your_model_path>/<your_model_name>_{:d}.pth'.format(max_epoch), map_location="<your_device>"))
```

***Note***:

- You need to replace `<your_model_path>`, `<your_model_name>`, and `<your_device>` with your own information.
- You need to use similar methods to save any other information required to continue the training, like the optimizer,
  the scheduler, activation function, etc.
- Please remove the saved model in time, coz they will take up a lot of space. Or you can write a script to remove the
  old models automatically.
- Please remember to stop the script after your work is done to avoid the waste of resources.

### Some notifications for the command you want to run

- Because directly running this script may only get your default Python environment, it's better to use the absolute
  path
  of your python interpreter and the script you want to run.
- Please also use the absolute path of the Python script you want to run.
- If you want to save the training logs, you can redirect the output to a file.

Here is an example:

```shell
./auto_gpu -t 0-01:00:00 -s -j <your_job_name> -a <your_csc_group_name> --cmd="/<your_anaconda_path>/anaconda3/envs/<your_env_name>/bin/python /<your_python_project_path>/<your_python_script_name>.py >> /<your_log_path>/<your_log_name>.log"
```

### Distributed Data Parallel (DDP) with PyTorch

This script is also available for Distributed Data Parallel (DDP) with PyTorch. You can apply for multiple nodes and
multiple GPUs on each node to train your model. There is already an excellent tutorial on how to use DDP on Puhti or
Mahti. Please find the
tutorial [here](https://tuni-itc.github.io/wiki/Technical-Notes/Distributed_dataparallel_pytorch/). Thanks to Soumya (
soumya.tripathy@tuni.fi).

## Some errors and solutions

- Wrong server

```shell
Please run this script on Puhti or Mahti.
```

- Invalid input, terminating...

```shell
Please check the input parameters and try again.
```

- The account is empty.

```shell
Please set the account you want to use.
```

- The command you want to run is empty.

```shell
Please set the command you want to run with double quotes.
```

- The job name is empty.

```shell
Please set the job name you want to use.
```

- mahti does not support the gpu_type 'gpu'.

```shell
Please choose the correct GPU type (small -s or medium -m) for Mahti.
```

- puhti does not support the gpu_type 'gpusmall' / 'gpumedium'.

```shell
Please choose the correct GPU type (gpu -l) for Puhti.
```

- Due to the resources limitation, please do not automatically apply for an interactive job, which will lead to lots of
  waste of resources. Please apply for such jobs manually.

```shell
Please run some non-interactive jobs.
```

- The format of time should be 'D-HH:MM:SS' (D: days, HH: hours, MM: minutes, SS: seconds)

```shell
Please set the time with the correct format.
```

- You have already applied for the job <job_name>, please use another job name.

```shell
Please set a different job name for different jobs.
```

- You have already applied for too many jobs, please stop some of them first.

```shell
Please stop some of the running jobs.
```

- Something wrong happened; please check if your request meets the rules of the server.

```shell
Please check the info on (https://docs.csc.fi/computing/running/batch-job-partitions/)
```

## Contact

If you have any questions or suggestions, please feel free to contact me, Shiqi Zhang (shiqi.zhang@tuni.fi).
