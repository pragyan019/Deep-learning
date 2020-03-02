## How to Train Your ResNet 1: Baseline
_The main innovations were mixed-precision training, choosing a smaller network with sufficient capacity for the task and employing higher learning rates to speed up stochastic gradient descent (SGD)._
### Mixed Precision Training

* A technique to train deep neural networks using half precision floating point numbers. 
* In this technique, weights, activations and gradients are stored in IEEE half-precision format.
* Half-precision floating numbers have limited numerical range compared to single-precision numbers.
*  Therefore two techniques are proposed to handle this loss of information :-
   1.  Firstly, maintaining a single-precision copy of the weights that accumulates the gradients after each optimizer step is recommended. This single-precision copy is rounded to half-precision format during training. 
   2.  Secondly, scaling the loss appropriately to handle the loss of information with half-precision gradients is proposed. 
* This approach works for a wide variety of models including convolution neural networks, recurrent neural networks and generative adversarial networks. 
* This technique works for large scale models with more than 100 million parameters trained on large datasets.
*  Using this approach, we can reduce the memory consumption of deep learning models by nearly 2x.

### Mitigating Preprocessing Bottlenecks

* Some of the image preprocessing (padding, normalisation and transposition) is needed on every pass through the training set and yet this work is being repeated each time.
*  Other preprocessing steps (random cropping and flipping) differ between epochs and it makes sense to delay applying these. 
*  Although the preprocessing overhead is being mitigated by using multiple CPU processes to do the work, it turns out that PyTorch dataloaders (as of version 0.4) launch fresh processes for each iteration through the dataset.
* The setup time for this is non-trivial, especially on a small dataset like CIFAR10.
*  By doing the common work once before training, removing pressure from the preprocessing jobs, we can reduce the number of processes needed to keep up with the GPU down to one.
*  In heavier tasks, requiring more preprocessing or feeding more than one GPU, an alternative solution could be to keep dataloader processes alive between epochs. 
*  In any case, the effect of removing the repeat work and reducing the number of dataloader processes is a further 15s saving in training time (almost half a second per epoch!) and a new training time of 308s.

* A bit more digging reveals that most of the remaining preprocessing time is spent calling out to random number generators to select data augmentations rather than in the augmentations themselves.
*  During a full training run we make several million individual calls to random number generators and by combining these into a small number of bulk calls at the start of each epoch we can shave a further 7s of training time.
*   Finally, at this point it turns out that the overhead of launching even a single process to perform the data augmentation outweighs the benefit and we can save a further 4s by doing the work on the main thread, leading to a final training time for today of 297s. 

## How to Train Your ResNet 2: Mini-batches

_Here we investigate mini-batch size and learn that we have a problem with forgetfulness_

* increasing batch sizes does not immediately lead to training instability as it should if curvature was the issue, but not if the issue is forgetfulness which should be mostly unaffected by batch size.
* the effects of curvature, which depends primarily on learning rate, from forgetfulness, which depends jointly on learning rate and dataset size. 
* for a dataset which is small enough such that forgetfulness is no longer an issue, learning rates should be pushed close to the limit allowed by curvature. For larger datasets, the optimal point can be significantly lower because of the forgetfulness effect.
* For a larger dataset such as ImageNet-1K, which consists of about 20× as many training examples as CIFAR10, the effects of forgetfulness are likely to be much more severe. This would explain why attempts to speed up training at small batch sizes with very high learning rates have failed on this dataset whilst [training with batches of size 8000](https://arxiv.org/abs/1706.02677) or more across multiple machines has been successful. At the very largest batch sizes, curvature effects dominate once again and for this reason there is substantial overlap between the techniques used in large batch training of ImageNet and fast single GPU training of CIFAR10.

## How to Train Your ResNet 3: Regularisation

* the default method of converting a model to half precision in PyTorch (as of version 0.4) triggers a slow code path which doesn’t use the optimized CuDNN routine. If we convert batch norm weights back to single precision then the fast code is triggered and things look much healthier
* A simple regularisation scheme that has been shown to be effective on CIFAR10 is so-called [Cutout](https://arxiv.org/abs/1708.04552) regularisation which consists of zeroing out a random subset of each training image.

## How to Train Your ResNet 4: Architecture

* The motivation for including residual blocks is to ease optimisation by creating shortcuts through the network. The hope is that the shorter paths represent shallow sub-networks which are relatively easy to train, whilst longer paths add capacity and computational depth. It seems reasonable to study how the shortest path through the network trains in isolation and to take steps to improve this before adding back the longer branches.
* Shorter training times and reduced model complexity should make the lower resource benchmarks easier to investigate and better optimised than their unconstrained cousins. Better optimised baselines in turn make it easier to accept or reject proposed changes.

## How to Train Your ResNet 5: Hyperparameters

_On hyperparameter tuning and how to avoid it._

* Changing λ and α holding  fixed.λα is equivalent to a change in initialisation scale of the weights (for parameters with scaling symmetry). Such changes in initialisation don’t change the loss and should have a diminishing effect on training over time if we’re lucky.
* It’s important for this argument that gradient updates lead (on average) to an increase in the scale of weights since otherwise there would be nothing to stop weight decay shrinking the weights to zero and de-stabilising the dynamics



