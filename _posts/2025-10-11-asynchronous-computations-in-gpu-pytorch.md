---
title: 'Understanding asynchronous computations in GPU in PyTorch'
date: 2025-10-11
permalink: posts/asynchronous-computations-gpu/
tags:
  - PyTorch
---

Understanding with a small experiment, the feature of pytorch of (almost) always having computations in GPU in an asynchronous fashion.

Here, I just try to run some computations (matrix multiplications) on a GPU in two ways. 

- In the first way, I just loop 1000 times and do the same matrix multiplication in GPU:0. At the end of each iteration, I run the command `torch.cuda.synchronize(device = gpu1)`, which synchronizes the kernels in the other GPU, GPU:1. There would actually be no kernels in GPU:1, but this command is still run to remove the bias that would otherwise be created due to the extra time torch.cuda.synchronize command would take in the second part.

- In the second way, I do that same thing, but instead of synchronizing GPU:1 at the end of each iteration, I synchronize GPU:0, which actually would have some meaningful effect, as that will lead to the frontend waiting for the kernels in GPU:0 to complete before the frontend moves on to the next commands in the code. This would likely cause some extra time consumed as now effectively we are doing serial processing, compared to the parallel (distributed over frontend, processing on cpu and backend, processing on gpu) processing in first part.

``` python
import torch
import time

gpu0 = torch.device('cuda:0')
gpu1 = torch.device('cuda:1')

# using cuda, with parallelization, but after each iteration, we synchronize the devices
starttime = time.time()
for _ in range(1000):
	a = torch.randn(size = (100,100), device = gpu0)
	b = torch.mm(a, a)
	torch.cuda.synchronize(device = gpu1)
torch.cuda.synchronize(device = gpu0)
endtime = time.time()
print(round(endtime - starttime, 3)) # 0.018

# making the operations serial by waiting for all kernels on all streams on the cuda0 device 
# to be completed at the end of each iteration
starttime = time.time()
for _ in range(1000):
	a = torch.randn(size = (100,100), device = gpu0)
	b = torch.mm(a, a)
	torch.cuda.synchronize(device = gpu0)
torch.cuda.synchronize(device = gpu0)
endtime = time.time()
print(round(endtime - starttime, 3)) # 0.021
```

So here we see a nice, consistent difference of 0.003 seconds. This difference is due to the fact that we are waiting for the kernels in the GPU:0 to complete at the end of each iteration in the second case, which is leading to some idle time for the frontend, and that is where those 0.003 seconds are going.

## References

- I originally got the idea of thinking about this in the form of frontend and backend from the section 13.2 of the book Dive Into Deep Learning (https://d2l.ai/).
