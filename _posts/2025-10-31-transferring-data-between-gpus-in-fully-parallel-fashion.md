---
title: 'Transferring data between GPUs in fully parallel fashion in PyTorch'
date: 2025-11-16
permalink: posts/transferring-data-between-gpus/
tags:
  - Hardware
  - PyTorch
---

Summarising results from my experiments in transferring data (pytorch tensors) between GPUs in a fully parallel fashion, i.e. non-blocking on host CPU, and also on the sender and receiver GPUs.

### Introduction 

In [this](https://docs.pytorch.org/tutorials/intermediate/pinmem_nonblock.html) pytorch tutorial, they explain many important concepts about memory storage on CPU, the asynchronicity of GPU operations in PyTorch, and transferring data between CPU and GPU (and how to make it parallelized). However, they don't touch upon the topic of making transfer between two GPUs parallelized. Here, I just talk about some experiments I did for the same and corresponding results (from pytorch profiler).

### Experiment Setup

The idea is simple: In an instance with 2 GPUs (lets call them gpu0 and gpu1), store a tensor (A) in gpu0, and another tensor (B) in gpu1. Then run the .to() method to transfer it to gpu1, and also run a command to multiply B by itself. If the transfer was blocking, the multiplication will happen just after the transfer finishes, else the multiplication and transfer will have some overlap in time.

First, make the necessary imports and initialization
``` python
# here, I try to test how I can transfer data from 1 gpu to another with it being non-blocking 
# on both the sender gpu and receiver gpu. This is especially different from the case of cpu 
# to gpu because the memory should be pinned in cpu for it to be nonblocking on the receiver, 
# but when its from gpu to gpu, its not possible as there is no concept of pinned memory in gpu.

# NOTE: One needs to have access to atleast two GPU devices for this code

import contextlib
import re
import torch
from torch.cuda import Stream
import time
import numpy

# make secondary streams on both gpus, which will be used for the transfer
s0 = Stream(device = torch.device('cuda', 0))
s1 = Stream(device = torch.device('cuda', 1))

torch.manual_seed(42)
A = torch.randn(1024**2 * 10, device="cuda:0")
B = torch.randn(1024**2 * 10, device="cuda:1")

# store in device variable, the cuda device onto which we will be making the transfer (receiver)
device = torch.device("cuda", 1)
```

Then, the function that we will profile. Simply, this function performs a transfer of a tensor from gpu0 to gpu1 and also tries to do a multiplication on gpu1. If the transfer and multiplication overlap in time, they were done in a parallelized fashion, else they weren't.
``` python
# The function we want to profile.
def inner(sender_streamed = False, receiver_streamed = False, non_blocking = True):
	# sender_streamed = False
	# receiver_streamed = True
	# non_blocking = True

	# according to values of arguments, we do the transfer in appropriate context
	if sender_streamed and receiver_streamed:
		with torch.cuda.stream(s0), torch.cuda.stream(s1):
			A_transferred = A.to(device, non_blocking = non_blocking) # having a separate argument for non_blocking as want to test if non-blocking effects cpu behaviour only, or effects gpus also in any direct way
        	secondary_stream0_event = s0.record_event() # create an event and record it (imagine record here like a screenshot, as it corresponds to a single timepoint) in secondary stream on gpu0 (s0), which can be used later for synchronization or timing purposes.
        	secondary_stream1_event = s1.record_event() # create an event and record it (imagine record here like a screenshot, as it corresponds to a single timepoint) in secondary stream on gpu1 (s1), which can be used later for synchronization or timing purposes.
        	# the purpose of creating these events is that I can synchronize them with cpu at the
        	# end of this function and ensure that profiling is done till both of them have been 
        	# reached which means that the transfer is done
    if sender_streamed and not receiver_streamed:
    	with torch.cuda.stream(s0):
			A_transferred = A.to(device, non_blocking = non_blocking) # having a separate argument for non_blocking as want to test if non-blocking effects cpu behaviour only, or effects gpus also in any direct way
        	secondary_stream0_event = s0.record_event() # create an event and record it (imagine record here like a screenshot, as it corresponds to a single timepoint) in secondary stream on gpu0 (s0), which can be used later for synchronization or timing purposes.
        	secondary_stream1_event = s1.record_event() # create an event and record it (imagine record here like a screenshot, as it corresponds to a single timepoint) in secondary stream on gpu1 (s1), which can be used later for synchronization or timing purposes.
        	# it doesn't really harm to have these two events in the two streams. If there was nothing queued on the stream, then the event will be reached superfast and there will be no time loss when we synchronize the cpu wrt it
    if not sender_streamed and receiver_streamed:
    	with torch.cuda.stream(s1):
			A_transferred = A.to(device, non_blocking = non_blocking) # having a separate argument for non_blocking as want to test if non-blocking effects cpu behaviour only, or effects gpus also in any direct way
        	secondary_stream0_event = s0.record_event() # create an event and record it (imagine record here like a screenshot, as it corresponds to a single timepoint) in secondary stream on gpu0 (s0), which can be used later for synchronization or timing purposes.
        	secondary_stream1_event = s1.record_event() # create an event and record it (imagine record here like a screenshot, as it corresponds to a single timepoint) in secondary stream on gpu1 (s1), which can be used later for synchronization or timing purposes.
        	# it doesn't really harm to have these two events in the two streams. If there was nothing queued on the stream, then the event will be reached superfast and there will be no time loss when we synchronize the cpu wrt it
    if not sender_streamed and not receiver_streamed:
    	with contextlib.nullcontext():
			A_transferred = A.to(device, non_blocking = non_blocking) # having a separate argument for non_blocking as want to test if non-blocking effects cpu behaviour only, or effects gpus also in any direct way
        	secondary_stream0_event = s0.record_event() # create an event and record it (imagine record here like a screenshot, as it corresponds to a single timepoint) in secondary stream on gpu0 (s0), which can be used later for synchronization or timing purposes.
        	secondary_stream1_event = s1.record_event() # create an event and record it (imagine record here like a screenshot, as it corresponds to a single timepoint) in secondary stream on gpu1 (s1), which can be used later for synchronization or timing purposes.
        	# it doesn't really harm to have these two events in the two streams. If there was nothing queued on the stream, then the event will be reached superfast and there will be no time loss when we synchronize the cpu wrt it

    # This operation can be executed during the GPU to GPU copy if and only if the 
    # the copy is done in the context of secondary streams in both gpus
    B_multiplied = B * B * B * B
    base_stream_event = torch.cuda.current_stream(device = device).record_event() # record an event in the default stream of the receiver gpu
    secondary_stream0_event.synchronize() # make the cpu thread wait till this event is completed
    secondary_stream1_event.synchronize() # make the cpu thread wait till this event is completed
    base_stream_event.synchronize() # make the cpu thread wait till this event is completed (this synchronization marks completion of multiplication task)
```

Then, the function which we use to call the `inner()` function within the context of pytorch profiler, so that we can see the time-trace of the different tasks in the different devices.
``` python
# Our profiler: profiles the `inner` function and stores the results in a .json file
def benchmark_with_profiler(sender_streamed = False, receiver_streamed = False, non_blocking = True) -> None:

    wait, warmup, active = 1, 1, 2 # the arguments for profiler schedule
    num_steps = wait + warmup + active
    rank = 0
    # we use the torch profiler's profile entrypoint as a context manager, and indicate which 
    # activities to profile, and how to schedule them.
    with torch.profiler.profile(
        activities=[
            torch.profiler.ProfilerActivity.CPU, # profile the activity of cpu
            torch.profiler.ProfilerActivity.CUDA, # profile the activity of cuda (gpu)
        ],
        # specify the number of wait, warmup, active steps in the schedule.
        schedule=torch.profiler.schedule(
            wait=wait, warmup=warmup, active=active, repeat=1, skip_first=1
        ),
    ) as prof:
        for step_idx in range(1, num_steps + 1):
            inner(sender_streamed, receiver_streamed, non_blocking)
            if rank is None or rank == 0:
                prof.step()
    prof.export_chrome_trace(f"trace_senderstreamed{int(sender_streamed)}_receiverstreamed{int(receiver_streamed)}_nonblocking{int(non_blocking)}.json")
```

Finally, call the `benchmark_with_profiler()` function for each combination of arguments.
``` python
# do the profiling of the inner function.
for i in [True, False]:
	for j in [True, False]:
		for k in [True, False]:
			benchmark_with_profiler(sender_streamed=i, receiver_streamed=j, non_blocking=k)	
```

### Results

When we look at the traces in `chrome://tracing/`, we get the following results.

Traces for first 3 conditions with non_blocking True shows no parallelization:
![first 3 conditions with non_blocking True](/images/traces_gpu_gpu_transfer1.png "Traces for first 3 conditions with non_blocking True")

With the transfer happening in the context of streams on both devices, we observe parallelization:
![with both streams, we observe parallelization](/images/traces_gpu_gpu_transfer_bothstreamed.png "With both streams, we observe parallelization")

Interestingly, when non_blocking = False also and both streams on, we observe parallelization:
![Both streams in case of non-blocking = False also gives parallelization](/images/traces_gpu_gpu_transfer_nonblocking_false.png "Both streams in case of non-blocking = False also gives parallelization")

### Conclusions

Hence, I was able to study about some intricacies of transferring data between GPUs in a parallelized fashion. We need to do the transfer in the context of secondary streams on both sender and receiver GPUs to be able to achieve parallelization in the transfer of data.

### References

- https://docs.pytorch.org/tutorials/intermediate/pinmem_nonblock.html
