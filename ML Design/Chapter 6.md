
# Data Parallelism

```python
import pandas as pd
import torch
from torch.utils.data import Dataset, DataLoader
from torchvision import transforms

DATA_DIR = "/kaggle/input/datasets/organizations/zalando-research/fashionmnist"

class FashionMNISTDataset(Dataset):

    def __init__(self, csv_file):
        self.data = pd.read_csv(csv_file)

        self.labels = torch.tensor(
            self.data.iloc[:, 0].values,
            dtype=torch.long
        ) ## all labels

        self.images = torch.tensor(
            self.data.iloc[:, 1:].values,
            dtype=torch.float32
        ).reshape(-1, 1, 28, 28) / 255.0 # all pixels reshaped

        self.images = (self.images - 0.2860) / 0.3530 # normalized

    def __len__(self):
        return len(self.labels)

    def __getitem__(self, idx):
        return self.images[idx], self.labels[idx] # for generator


train_dataset = FashionMNISTDataset(
    f"{DATA_DIR}/fashion-mnist_train.csv"
)
```

```python
#model
import torch.nn as nn

class CNN(nn.Module):
    def __init__(self):
        super().__init__()

        self.features = nn.Sequential(
            nn.Conv2d(1,64,3,padding=1), 
            # IN CHANNEL, OUT CHANNELS(NO. OF FILTERS),KERNEL_SIZE
            nn.ReLU(),

            nn.Conv2d(64,128,3,padding=1),
            nn.ReLU(),

            nn.MaxPool2d(2),

            nn.Conv2d(128,256,3,padding=1),
            nn.ReLU(),

            nn.Conv2d(256,256,3,padding=1),
            nn.ReLU(),

            nn.MaxPool2d(2),

            nn.Conv2d(256,512,3,padding=1),
            nn.ReLU(),

            nn.AdaptiveAvgPool2d((1,1))
        )

        self.classifier = nn.Sequential(
            nn.Flatten(),
            nn.Linear(512,256),
            nn.ReLU(),
            nn.Linear(256,10)
        )

    def forward(self,x):
        x = self.features(x)
        return self.classifier(x)

```

```python

import time

model.train() #sets model to training mode

torch.cuda.synchronize()

start = time.perf_counter() #precision counter

for epoch in range(10):
    for images , labels in train_loader:
        images = images.to(
            device,non_blocking =True # async
        ) # transfer images to GPU

        labels = labels.to(
            device,non_blocking=True
        ) # transfer labels to GPU

        optimizer.zero_grad(
            set_to_none=True
        ) #clear the gradients in the parameters

        outputs = model(images) # forward pass

        loss = criterion(
            outputs,
            labels
        ) # compute loss

        loss.backward() # backward pass

        optimizer.step() # update the parameters with gradients

torch.cuda.synchronize()

elapsed = time.perf_counter() - start
print(f"Training time: {elapsed:.2f} seconds") ## 180 SECS

```


using single gpu , the training takes 180 secs

## DDP - Distributed Data Parallel

```python
%%writefile train_ddp.py

import os
import time
import pandas as pd

import torch.distributed as dist

from torch.utils.data import Dataset, DataLoader
from torch.utils.data.distributed import DistributedSampler
from torch.nn.parallel import DistributedDataParallel as DDP



# -----------------------------
# DDP setup
# -----------------------------

local_rank = int(os.environ["LOCAL_RANK"]) # get the gpu number from while running

torch.cuda.set_device(local_rank)

device = torch.device(
    f"cuda:{local_rank}"
)

dist.init_process_group(
    backend="nccl"
) # intialies process group for communication


# -----------------------------
# Dataset
# -----------------------------

train_dataset = FashionMNISTDataset(
    f"{DATA_DIR}/fashion-mnist_train.csv"
)

sampler = DistributedSampler(
    train_dataset,
    shuffle=True 
) # this is essential to separate training samples btw gpu's models

train_loader = DataLoader(
    train_dataset,

    # IMPORTANT:
    # 128 on each GPU
    batch_size=128,

    sampler=sampler,

    num_workers=2,
    pin_memory=True
)


# -----------------------------
# Model
# -----------------------------

model = CNN().to(device)

model = DDP(
    model,
    device_ids=[local_rank]
) # we need to wrap it in DDP


criterion = nn.CrossEntropyLoss()

optimizer = torch.optim.Adam(
    model.parameters(),
    lr=0.001
)


# -----------------------------
# Training
# -----------------------------

dist.barrier() # for syncing both models

if local_rank == 0:
    start = time.perf_counter()


model.train()

for epoch in range(10):

    # Important:
    # changes the shuffle order each epoch
    sampler.set_epoch(epoch)

    for images, labels in train_loader:

        images = images.to(
            device,
            non_blocking=True
        )

        labels = labels.to(
            device,
            non_blocking=True
        )

        optimizer.zero_grad(
            set_to_none=True
        )

        outputs = model(images)

        loss = criterion(
            outputs,
            labels
        )

        loss.backward()

        optimizer.step() #sync gradients using allreduce
        # so both model is same after every update


# Make sure both GPUs finished
dist.barrier()

if local_rank == 0:

    elapsed = time.perf_counter() - start

    total_images = len(train_dataset) * 10

    print(
        f"\nTraining time: {elapsed:.2f} seconds"
    )

    print(
        f"Throughput: "
        f"{total_images / elapsed:.2f} images/sec"
    )


dist.destroy_process_group() # this takes half , if twice gpu used , and 1/n th time if n gpus used 
```


```
                 Training dataset
                       │
             DistributedSampler
                  ↙         ↘
             Process 0     Process 1
                │             │
             GPU 0          GPU 1
                │             │
            Model 0        Model 1
                │             │
           batch 0         batch 1
                │             │
          forward/backward
                │             │
                └─────┬───────┘
                      ↓
                 All-Reduce
                 gradients
                      ↓
             same model weights
```


the gpus get its own model , and distributed sampler handles splitting the original data into unique batches so both model learn from new samples each epoch
and after gradients computed on both model ,
they will not be same , so we use algorithms like Ring-AllReduce to sync gradients 
now both models gradients across all parameters are same ,so after every update the both models are in same state

the core condition when we can use this is a model we have can fit into a gpu , and we have lots of gpu so we can speed this up but having the model's copy in each gpu and distribute the data

but this becomes its drawback , `each model has to be loaded into all the gpu we'll be using` 


# Model Parallel
when we cant load the whole model in the same GPU, we split the model into different parts and load those parts in different , so for example

```
MODEL A

PART 1 -> Layer1, 2 ,3 , ... 10 -> GPU0
PART 2 -> Layer 11, ... , 20 -> GPU1
..
..
```

a drawback in this approach is that this only makes us use all the GPUs we have to be able to train a model but not in a effective way because each GPU only holds a part so when GPU 0 is computing its forward pass all other GPU's are idle so we are not utilizing all the GPU's to its full capacity 
also each gpu needs its part weights , optimizer , activations states loaded into the model


```python
class HugeModel(nn.Module):

    def __init__(self):
        super().__init__()

        self.layers = nn.Sequential(
            nn.Linear(32768, 32768),
            nn.ReLU(),
            
            nn.Linear(32768, 32768),
            nn.ReLU(),
        )

    def forward(self, x):
        return self.layers(x)


# Create on CPU first
model = HugeModel()

params = sum(p.numel() for p in model.parameters())

print(f"Parameters: {params:,}")
print(f"FP32 model size: {params * 4 / 1024**3:.2f} GB")
```

this model will load into a 16 gb memory , but it cant train because it still needs to load the optimizer, activations, and data 


so 

```python
class ModelParallel(nn.Module):

    def __init__(self):
        super().__init__()

        # First half → GPU 0
        self.part1 = nn.Sequential(

            nn.Linear(32768, 32768),
            nn.ReLU(),

        ).to("cuda:0")


        # Second half → GPU 1
        self.part2 = nn.Sequential(

            nn.Linear(32768, 32768),
            nn.ReLU(),

        ).to("cuda:1")


    def forward(self, x):

        # Input goes to GPU 0
        x = x.to("cuda:0")

        # First half of model
        x = self.part1(x)

        # Move activation GPU 0 → GPU 1
        x = x.to("cuda:1")

        # Second half of model
        x = self.part2(x)
        
        return x
```

look how each part is loaded into different GPU, and in forward , see how the part 1 has to be computed before part 2 can take over this creates the idle problem

# Pipeline Parallelism
this is to solve the problem in model parallel i.e, the idle time 
this creates a micro batches of the batch and processes them in a pipeline format

```
Model parallel
Time →

GPU 0:  ████████████████              ████████████████
        Part 1                        Part 1

GPU 1:                  ████████████████              ████████████████
                        Part 2                        Part 2
```


```
pipeline parallelism
Time ──────────────────────────────────────────→

GPU 0
MB1       █████
MB2           █████
MB3               █████
MB4                   █████

GPU 1
MB1           █████
MB2               █████
MB3                   █████
MB4                       █████
```

```python
%%writefile pipeline_train.py

import os
import time

import torch
import torch.nn as nn
import torch.distributed as dist

from torch.distributed.pipelining import (
    pipeline, # responsible for creating the pipeline
    SplitPoint, # responsible for handling what part goes to which GPU
    ScheduleGPipe, # responsible for how to do forward and backwar pass 
)


# ============================================================
# Model
# ============================================================

class Model(nn.Module):

    def __init__(self):
        super().__init__()

        self.part1 = nn.Sequential(
            nn.Linear(28000, 28000),
            nn.ReLU(),

            nn.Linear(28000, 28000),
            nn.ReLU(),
        )

        self.part2 = nn.Sequential(
            nn.Linear(28000, 28000),
            nn.ReLU(),

            nn.Linear(28000, 28000),
            nn.ReLU(),
        )

    def forward(self, x):

        x = self.part1(x)
        x = self.part2(x)

        return x


# ============================================================
# DDP / pipeline process setup
# ============================================================

rank = int(os.environ["LOCAL_RANK"])

device = torch.device(f"cuda:{rank}")

torch.cuda.set_device(rank)

dist.init_process_group(
    backend="nccl",
    device_id=device
)


# ============================================================
# Model
# ============================================================

model = Model()


# ============================================================
# Build pipeline
# ============================================================

example_input = torch.randn(
    512,
    28000
) # just a reference for pipeline to create the schedule

pipe = pipeline(
    module=model,
    mb_args=(example_input,),
    split_spec={
        "part2": SplitPoint.BEGINNING
    }
)


# ============================================================
# Create this rank's pipeline stage
# ============================================================

stage = pipe.build_stage(
    stage_index=rank,
    device=device
)


# ============================================================
# Optimizer
# ============================================================

optimizer = torch.optim.SGD(
    stage.submod.parameters(), # we only load the required params for that gpu
    lr=0.001
)


# ============================================================
# Input
# ============================================================

if rank == 0:

    x = torch.randn(
        2048,
        28000,
        device=device
    )

else:

    x = None


# ============================================================
# Pipeline schedule
# ============================================================

schedule = ScheduleGPipe(
    stage,
    n_microbatches=4 
)


# ============================================================
# Training
# ============================================================

dist.barrier()

if rank == 0:
    start = time.perf_counter()


for step in range(10):

    optimizer.zero_grad(
        set_to_none=True
    )

    if rank == 0:

        # Pass the complete batch.
        # ScheduleGPipe handles the microbatching.
        schedule.step(x)

    else:

        schedule.step()


dist.barrier()

torch.cuda.synchronize()

if rank == 0:

    elapsed = time.perf_counter() - start

    print(
        f"\nPipeline training time: "
        f"{elapsed:.2f} seconds"
    )


dist.destroy_process_group()
```

something like this to attempt pipeline parallelism




