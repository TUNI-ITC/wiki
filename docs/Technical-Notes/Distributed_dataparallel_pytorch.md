# Distributed data parallel training using Pytorch on the multiple nodes of CSC and Narvi clusters

## Table of Contents

1.  [Motivation](#motivation)
2.  [Outline](#outline)
3.  [Setting up a PyTorch model without DistributedDataParallel](#setting-up-a-pytorch-model-without-distributeddataparallel)
4.  [Setting up the same model with DistributedDataParallel](#setting-up-the-same-model-with-distributeddataparallel)
5.  [DistributedDataParallel as a Batch job in the servers](#distributeddataparallel-as-a-batch-job-in-the-servers)
6.  [Tips and Tricks](#tips-and-tricks)
7.  [Acknowledgements](#acknowledgements)



## Motivation

Training a Deep Neural Network (DNNs) is notoriously time-consuming especially nowadays when they are getting bigger to get better. To reduce the training time, we mostly train it on the multiple gpus within a single node or across different nodes. This tutorial is focused on the latter where multiple nodes are utilised using PyTorch. Although there are many tutorials available on the web including one from the [PyTorch](https://pytorch.org/tutorials/intermediate/ddp_tutorial.html), they are not self-sufficient in explaining some of the key issues like how to run the code, how to save checkpoints, or how to create a batch script for this in the severs. I have given a starter kit here which addresses these issues and can be helpful to students of our university in setting up their first multi-gpu training in the servers like CSC-Puhti or Narvi.


## Outline

PyTorch mostly provides two functions namely `nn.DataParallel` and `nn.DistributedDataParallel` to use multiple gpus in a single node and multiple nodes during the training respectively. However, it is recommended by [PyTorch](https://pytorch.org/tutorials/intermediate/ddp_tutorial.html) to use `nn.DistributedDataParallel` even in the single node to train faster than the `nn.DataParallel`. For more details, I would recommend reading the PyTorch docs. This tutorial assumes that the reader is familiar with the DNNs training using PyTorch and basic operations on the gpu-servers of our university.


## Setting up a PyTorch model without DistributedDataParallel

I have considered a simple Auto-Encoder (AE) model for demonstration where the inputs are images of digits from [MNIST](http://yann.lecun.com/exdb/mnist/) data-set. Just to be clear, AE takes images as input and encodes it to a much smaller dimension w.r.t its inputs and then try to reconstruct the images back from those smaller dimensions. It can be considered as a process of compression and decompression. We train the network to learn this smaller dimension such that the reconstructed image is very close to input. Let's begin by defining the network structure.

``` python
import torch
import torch.nn as nn
import torchvision
from argparse import ArgumentParser

class AE(nn.Module):
    def __init__(self, **kwargs):
        super().__init__()

        self.net = nn.Sequential(
            nn.Linear(in_features=kwargs["input_shape"], out_features=128),
            nn.ReLU(inplace=True),
            # small dimension
            nn.Linear(in_features=128, out_features=128),
            nn.ReLU(inplace=True),
            nn.Linear(in_features=128, out_features=128),
            nn.ReLU(inplace=True),
            # Recconstruction of input
            nn.Linear(in_features=128, out_features=kwargs["input_shape"]),
            nn.ReLU(inplace=True)
        )

    def forward(self, features):
        reconstructed = self.net(features)
        return reconstructed
```
Lets create a `train()` function where we load the MNIST data-set and this can easily be done from the `torchvision.dataset` library as follows
``` python
def train(gpu, args):
    transform = torchvision.transforms.Compose([
        torchvision.transforms.ToTensor()
    ])

    train_dataset = torchvision.datasets.MNIST(
        root="~/mnist_dataset", train=True, transform=transform, download=True
    )

    train_loader = torch.utils.data.DataLoader(
        train_dataset, batch_size=128, shuffle=True, num_workers=4,
        pin_memory=True
    )
```
Transfer the model to the GPU now and declare the optimiser and loss criterion for the training process.
``` python
def train(gpu, args):
    transform = torchvision.transforms.Compose([
        torchvision.transforms.ToTensor()
    ])

    train_dataset = torchvision.datasets.MNIST(
        root="./mnist_dataset", train=True, transform=transform, download=True
    )

    train_loader = torch.utils.data.DataLoader(
        train_dataset, batch_size=128, shuffle=True, num_workers=4,
        pin_memory=True
    )

    # load the model to the specified device, gpu-0 in our case
    model = AE(input_shape=784).cuda(gpu)
    # create an optimizer object
    # Adam optimizer with learning rate 1e-3
    optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)
    # Loss function
    criterion = nn.MSELoss()
```
Now wrap everything in the training function and start training

``` python
def train(gpu, args):
    transform = torchvision.transforms.Compose([
        torchvision.transforms.ToTensor()
    ])

    train_dataset = torchvision.datasets.MNIST(
        root="./mnist_dataset", train=True, transform=transform, download=True
    )

    train_loader = torch.utils.data.DataLoader(
        train_dataset, batch_size=128, shuffle=True, num_workers=4,
        pin_memory=True
    )

    # load the model to the specified device, gpu-0 in our case
    model = AE(input_shape=784).cuda(gpu)
    # create an optimizer object
    # Adam optimizer with learning rate 1e-3
    optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)
    # Loss function
    criterion = nn.MSELoss()

    for epoch in range(args.epochs):
        loss = 0
        for batch_features, _ in train_loader:
            # reshape mini-batch data to [N, 784] matrix
            # load it to the active device
            batch_features = batch_features.view(-1, 784).cuda(gpu)

            # reset the gradients back to zero
            # PyTorch accumulates gradients on subsequent backward passes
            optimizer.zero_grad()

            # compute reconstructions
            outputs = model(batch_features)

            # compute training reconstruction loss
            train_loss = criterion(outputs, batch_features)

            # compute accumulated gradients
            train_loss.backward()
            # pe-rform parameter update based on current gradients
            optimizer.step()

            # add the mini-batch training loss to epoch loss
            loss += train_loss.item()

            # compute the epoch training loss
        loss = loss / len(train_loader)

        # display the epoch training loss
        print("epoch: {}/{}, loss = {:.6f}".format(epoch+1, args.epochs, loss))
```
Now lets finish this code with a `main()` function that calls the train function and defines the required arguments.
``` python
def main():
    parser = ArgumentParser()
    parser.add_argument('--ngpus', default=1, type=int,
                        help='number of gpus per node')

    parser.add_argument('--epochs', default=2, type=int, metavar='N',
                        help='number of total epochs to run')
    args = parser.parse_args()
    train(0, args)

if __name__ == '__main__':
    main()
```

## Setting up the same model with DistributedDataParallel

With the multiprocessing, we will run our training script in each node separately and ask PyTorch to handle the synchronisation between them. It makes sure that in each iteration, the same network weights are present in every node but use different data for the forward pass. Then the gradients are accumulated from every node to calculate the change in weights which will be sent to each node for the update. In short, the same network operates on different data in different nodes in parallel to make things faster. To let this internal communication happen between the nodes, we need few information to setup the DistributedParallel environment such as 1. how many nodes we are using, 2. what is the ip-address of the master node and 3. The number of gpus in a single node. I have changed the order of the above code to make it more understandable. We will first start from the `main` function by defining all the necessary variables.

-   A single node can be understood as a single computer with its own gpus and cpus. Here we need multiple of such computers. One thing to remember is that these nodes should be connected to each other. In the servers, they are always connected to each other so we can use it without any problems. In the script, we need to mention the ip-address and port of one of the nodes (we call it the master node) so that all other nodes can be connected to that automatically when we start the script in those nodes.

``` python
import torch
import torch.nn as nn
import torchvision
import torch.multiprocessing as mp
import torch.distributed as dist
from argparse import ArgumentParser
import os

if __name__ == "__main__":

    parser = ArgumentParser()
    parser.add_argument('--nodes', default=1, type=int)
    parser.add_argument('--local_ranks', default=0, type=int,
                        help="Node's order number in [0, num_of_nodes-1]")
    parser.add_argument('--ip_adress', type=str, required=True,
                        help='ip address of the host node')
    parser.add_argument("--checkpoint", default=None,
                        help="path to checkpoint to restore")
    parser.add_argument('--ngpus', default=1, type=int,
                        help='number of gpus per node')
    parser.add_argument('--epochs', default=2, type=int, metavar='N',
                        help='number of total epochs to run')

    args = parser.parse_args()
    # Total number of gpus availabe to us.
    args.world_size = args.ngpu * args.nodes
    # add the ip address to the environment variable so it can be easily avialbale
    os.environ['MASTER_ADDR'] = args.ip_adress
    print("ip_adress is", args.ip_adress)
    os.environ['MASTER_PORT'] = '8888'
    os.environ['WORLD_SIZE'] = str(args.world_size)
    # nprocs: number of process which is equal to args.ngpu here
    mp.spawn(train, nprocs=args.ngpus, args=(args,))
```

-   You can imagine the `local_rank` as an unique number associated to each node starting from zero to number of nodes-1. We assign zero rank to the node whose ip-address is passed to the `main()` and we start the script first on that node. Further, we are going use this number to calculate one more rank for each gpu in that node.
-   Instead of calling the `train` function once, we spawn `args.ngpus` processes in each node to run `args.ngpus` instances of `train` function in parallel.

Now lets define the function `train` that can handle these multiple processes.

``` python
def train(gpu, args):

    args.gpu = gpu
    print('gpu:',gpu)
    # rank calculation for each process per gpu so that they can be identified uniquely.
    rank = args.local_ranks * args.ngpus + gpu
    print('rank:',rank)
    # Boilerplate code to initialize the parallel prccess.
    # It looks for ip-address and port which we have set as environ variable.
    # If you don't want to set it in the main then you can pass it by replacing
    # the init_method as ='tcp://<ip-address>:<port>' after the backend.
    # More useful information can be found in
    # https://yangkky.github.io/2019/07/08/distributed-pytorch-tutorial.html

    dist.init_process_group(
        backend='nccl',
        init_method='env://',
        world_size=args.world_size,
        rank=rank
    )
    torch.manual_seed(0)
    # start from the same randomness in different nodes. If you don't set it
    # then networks can have different weights in different nodes when the
    # training starts. We want exact copy of same network in all the nodes.
    # Then it will progress from there.

    # set the gpu for each processes
    torch.cuda.set_device(args.gpu)


    transform = torchvision.transforms.Compose([
        torchvision.transforms.ToTensor()
    ])

    train_dataset = torchvision.datasets.MNIST(
        root="~/mnist_dataset", train=True, transform=transform, download=True
    )
    # Ensures that each process gets differnt data from the batch.
    train_sampler = torch.utils.data.distributed.DistributedSampler(
        train_dataset, num_replicas=args.world_size, rank=rank
    )

    train_loader = torch.utils.data.DataLoader(
        train_dataset,
        # calculate the batch size for each process in the node.
        batch_size=int(128/args.ngpus),
        shuffle=(train_sampler is None),
        num_workers=4,
        pin_memory=True,
        sampler=train_sampler
    )
```

-   As we are going to submit the training script to each node separately, we need to set a random seed to fix the randomness involved in the code. For example, in the very first iteration the network weights will start from the same random weights (seed=0) in the different nodes. Then PyTorch will handle the synchronisation and at the end of training, we will have the same network weights in each node.
-   `train_sampler`, `manual_seed` and `modified batch size in the dataloader` are important steps to remember while setting this up.

Finally, wrap the model as DistributedDataParallel and start the training.
``` python
def train(gpu, args):
    args.gpu = gpu
    print('gpu:',gpu)
    rank = args.local_ranks * args.ngpus + gpu
    # rank calculation for each process per gpu so that they can be
    # identified uniquely.
    print('rank:',rank)
    # Boilerplate code to initialise the parallel process.
    # It looks for ip-address and port which we have set as environ variable.
    # If you don't want to set it in the main then you can pass it by replacing
    # the init_method as ='tcp://<ip-address>:<port>' after the backend.
    # More useful information can be found in
    # https://yangkky.github.io/2019/07/08/distributed-pytorch-tutorial.html

    dist.init_process_group(
        backend='nccl',
        init_method='env://',
        world_size=args.world_size,
        rank=rank
    )
    torch.manual_seed(0)
    # start from the same randomness in different nodes.
    # If you don't set it then networks can have different weights in different
    # nodes when the training starts. We want exact copy of same network in all
    # the nodes. Then it will progress form there.

    # set the gpu for each processes
    torch.cuda.set_device(args.gpu)


    transform = torchvision.transforms.Compose([
        torchvision.transforms.ToTensor()
    ])

    train_dataset = torchvision.datasets.MNIST(
        root="./mnist_dataset", train=True, transform=transform, download=True
    )
    # Ensures that each process gets differnt data from the batch.
    train_sampler = torch.utils.data.distributed.DistributedSampler(
        train_dataset, num_replicas=args.world_size, rank=rank
    )

    train_loader = torch.utils.data.DataLoader(
        train_dataset,
        # calculate the batch size for each process in the node.
        batch_size=int(128/args.ngpus),
        shuffle=(train_sampler is None),
        num_workers=4,
        pin_memory=True,
        sampler=train_sampler
    )


    # load the model to the specified device, gpu-0 in our case
    model = AE(input_shape=784).cuda(args.gpus)
    model = torch.nn.parallel.DistributedDataParallel(
        model_sync, device_ids=[args.gpu], find_unused_parameters=True
    )
    # create an optimizer object
    # Adam optimizer with learning rate 1e-3
    optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)
    # Loss function
    criterion = nn.MSELoss()

    for epoch in range(args.epochs):
        loss = 0
        for batch_features, _ in train_loader:
            # reshape mini-batch data to [N, 784] matrix
            # load it to the active device
            batch_features = batch_features.view(-1, 784).cuda(args.gpus)

            # reset the gradients back to zero
            # PyTorch accumulates gradients on subsequent backward passes
            optimizer.zero_grad()

            # compute reconstructions
            outputs = model(batch_features)

            # compute training reconstruction loss
            train_loss = criterion(outputs, batch_features)

            # compute accumulated gradients
            train_loss.backward()

            # perform parameter update based on current gradients
            optimizer.step()

            # add the mini-batch training loss to epoch loss
            loss += train_loss.item()

        # compute the epoch training loss
        loss = loss / len(train_loader)

        # display the epoch training loss
        print("epoch: {}/{}, loss = {:.6f}".format(epoch+1, args.epochs, loss))
        if rank == 0:
            dict_model = {
                'state_dict': model.state_dict(),
                'optimizer': optimizer.state_dict(),
                'epoch': args.epochs,
            }
            torch.save(dict_model, './model.pth')
```

-   Save the model only when the rank is zero because all the models are the same. We only need to save one copy of the model. If we are not careful here then all the processes will try to save weights and can corrupt the weights.

Save the script as `train.py` in the CSC or Narvi server and submit an interactive job with two gpu nodes (Lets quickly test it on `gputest` node as `srun --pty --account=Project_** --nodes=2 -p gputest --gres=gpu:v100:1,nvme:100 -t 00:15:00 --mem-per-cpu=20000 --ntasks-per-node=1 --cpus-per-task=8 /bin/bash -i`). Once it is allocated, ssh to each node in two terminals as `ssh <node name>`) and submit the job by typing `python train.py --ip_adress=**.**.**.** --nodes 2 --local_rank 0 --ngpus 1 --epochs 1` and `python train.py --ip_adress=<same as the first> --nodes 2 --local_rank 1 --ngpus 1 --epochs 1` to each of them respectively. Two job should start with synchronisation and training will begin soon after.

-   The ip-address of a node can be obtained by `ping <node name>`



## DistributedDataParallel as a Batch job in the servers

When we are submitting the interactive jobs, we know the exact node name and can obtain the ip-address for that beforehand. However, in the batch job, it needs to be programmed to automate most of the stuff. We have to make minimum changes to the existing code and write a `.sh` script to submit the job. Our `train.py` script are modified only in the first few lines of the `train()` function as follows

<!-- If you change this block make sure `hl_lines` points to the correct place -->
``` python hl_lines="8"
def train(gpu, args):

    args.gpu = gpu
    print('gpu:',gpu)

    # rank calculation for each process per gpu so that they can be
    # identified uniquely.
    rank = int(os.environ.get("SLURM_NODEID")) * args.ngpus + gpu
    print('rank:',rank)
    # Boilerplate code to initialise the parallel process.
    # It looks for ip-address and port which we have set as environ variable.
    # If you don't want to set it in the main then you can pass it by replacing
    # the init_method as ='tcp://<ip-address>:<port>' after the backend.
    # More useful information can be found in
    # https://yangkky.github.io/2019/07/08/distributed-pytorch-tutorial.html

    dist.init_process_group(
        backend='nccl',
        init_method='env://',
        world_size=args.world_size,
        rank=rank
    )
    torch.manual_seed(0)
    # start from the same randomness in different nodes.
    # If you don't set it then networks can have differnt weights in different
    # nodes when the training starts. We want exact copy of same network in
    # all the nodes. Then it will progress form there.

    # set the gpu for each processes
    torch.cuda.set_device(args.gpu)
```

-   Instead of using local rank in calculation of process rank, we use environment variable `$SLURM_NODEID` which is unique for each slurm node.

Keeping everything else in the code same, now lets write the batch script for CSC-puhti. Same script can be used for Narvi.

``` bash
#!/bin/bash
#SBATCH --job-name=name
#SBATCH --account=Project_******
#SBATCH -o out.txt
#SBATCH -e err.txt
#SBATCH --partition=gpu
#SBATCH --time=08:00:00
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=4
#SBATCH --mem-per-cpu=8000
#SBATCH --gres=gpu:v100:4
#SBATCH  --nodes=2
module load gcc/8.3.0 cuda/10.1.168
source <virtual environment name>

# if some error happens in the initialation of parallel process then you can
# get the debug info. This can easily increase the size of out.txt.
export NCCL_DEBUG=INFO  # comment it if you are not debugging distributed parallel setup

export NCCL_DEBUG_SUBSYS=ALL # comment it if you are not debugging distributed parallel setup

# find the ip-address of one of the node. Treat it as master
ip1=`hostname -I | awk '{print $2}'`
echo $ip1

# Store the master node’s IP address in the MASTER_ADDR environment variable.
export MASTER_ADDR=$(hostname)

echo "r$SLURM_NODEID master: $MASTER_ADDR"

echo "r$SLURM_NODEID Launching python script"

srun python train.py --nodes=2 --ngpus 4 --ip_adress $ip1 --epochs 1
```
!!! warning "To Narvi users"
	Change the ip1=`hostname -I | awk '{print $2}'` line to ip1=`hostname -I | awk '{print $1}'` to correctly parse the ip address.
!!! warning "To Mahti users"
	For now add `export NCCL_IB_DISABLE=1` to the batch script to prevent the occasional hang in the training loop. However, I am not sure whether this is happening becasue of Mahti or Pytorch 1.8.
## Tips and Tricks

-   If you have `os.mkdir` inside the script then always wrap it with `try and except`. Multiple processes will try to create a new folder and they will throw errors that the directory already exists.
-   When resuming the network weights if your model complains that the tensors are not on the same advice and points to the optimiser then it is mostly caused by this [optimizer-error](https://github.com/pytorch/pytorch/issues/2830). Just add these few lines after loading the optimizer from the checkpoints.

``` python
for state in optimizer.state.values():
    for k, v in state.items():
        if isinstance(v, torch.Tensor):
            state[k] = v.cuda(gpus)
```

-   To run on a single node with multiple gpus, just make the `--nodes=1` in the batch script.
-   If you Batchnorm\*d inside the network then you may consider replacing them with `sync-batchnorm` to have better batch statistics while using DistributedDataParallel.
-   Use this feature when it is required to optimise the gpu usage.


## Acknowledgements

I found this [article](https://yangkky.github.io/2019/07/08/distributed-pytorch-tutorial.html) really helpful when I was setting up my DistributedDataParallel framework. Many missing details can be found in this article which is skipped here to focus more on the practical things. If you have any suggestions then reach me at `soumya.tripathy@tuni.fi`.
