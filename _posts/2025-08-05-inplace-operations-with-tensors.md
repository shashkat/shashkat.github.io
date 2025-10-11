---
title: 'Notes about in-place operations on tensors in PyTorch'
date: 2025-08-05
permalink: posts/inplace-operations/
tags:
  - PyTorch
---

Notes about some possible in-place operation cases with tensors in pytorch.

- Following is the base code for which there are many possible cases and the results of in-place operations for each are summarised below:

``` python
leaf = torch.tensor([1,2,3,4,5], dtype = torch.float, requires_grad = True)
middle = leaf + 2 # creating an intermediate tensor because can't do at all, in-place operations on leaf tensors requiring grad

# change middle in-place
middle.multiply_(3) # OR middle[2] = 10

# get root from middle
root = middle*2 # OR root = middle.pow(2) OR root = middle.exp()

# finish up with calling .backward() and observing leaf.grad
root.sum().backward()
leaf.grad
```

| Sr No. | Operation on Middle Tensor    | Inplace Modification                      | Inplace Modification When            | Can Call Backward on Root? | Result                                           |
|--------|------------------------------|-------------------------------------------|--------------------------------------|----------------------------|--------------------------------------------------|
| 1      | Any                          | None                                      | NA                                   | Yes                        | Standard operation without inplace modification   |
| 2      | middle*2                     | .add_(), .multiply(), etc.                | after computing root from middle     | Yes                        | gradients same as without inplace operation       |
| 3      | middle*2                     | inplace modification (e.g., tensor[2]=10) | after computing root from middle     | Yes                        | gradients same as without inplace operation       |
| 4      | middle.pow(2)                | .add_(), .multiply(), etc.                | after computing root from middle     | No                         | NA                                               |
| 5      | middle.pow(2)                | inplace modification (e.g., tensor[2]=10) | after computing root from middle     | No                         | NA                                               |
| 6      | middle.exp()                 | .add_(), .multiply(), etc.                | after computing root from middle     | Yes                        | gradients same as without inplace operation       |
| 7      | middle.exp()                 | inplace modification (e.g., tensor[2]=10) | after computing root from middle     | Yes                        | gradients same as without inplace operation       |
| 8      | middle*2                     | .add_(), .multiply(), etc.                | before computing root from middle    | Yes                        | gradients same as without inplace operation       |
| 9      | middle*2                     | inplace modification (e.g., tensor[2]=10) | before computing root from middle    | Yes                        | Modified value removed from computation graph     |
| 10     | middle.pow(2)                | .add_(), .multiply(), etc.                | before computing root from middle    | Yes                        | gradients same as without inplace operation       |
| 11     | middle.pow(2)                | inplace modification (e.g., tensor[2]=10) | before computing root from middle    | Yes                        | Modified value removed from computation graph     |
| 12     | middle.exp()                 | .add_(), .multiply(), etc.                | before computing root from middle    | Yes                        | gradients same as without inplace operation       |
| 13     | middle.exp()                 | inplace modification (e.g., tensor[2]=10) | before computing root from middle    | Yes                        | Modified value removed from computation graph     |

The cases 9,11,13 are common apparently. In the masked softmax function, we make the entries in the attention-weight matrix which are beyond valid_lens to be a large negative number, so that upon exponentiation (which happens in softmax), those entries become very small in the attention-weight matrix. Something like this:

``` python
def _sequence_mask(X, valid_len, value=0):
  maxlen = X.size(1) # this is num_keys, as X passed into this func is in 2d form (num_queries*num_batches, num_keys)
  mask = torch.arange((maxlen), dtype=torch.float32, device=X.device)[None, :] < valid_len[:, None]
  X[~mask] = value
  return X
```
