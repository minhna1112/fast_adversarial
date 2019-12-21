# Fast adversarial training using FGSM

*A repository that implements the fast adversarial training code using an FGSM adversary, capable of training a robust CIFAR10 classifier in 6 minutes and a robust ImageNet classifier in 12 hours. Created by [Eric Wong](https://riceric22.github.io), [Leslie Rice](https://leslierice1.github.io/), and [Zico Kolter](http://zicokolter.com). See our paper on arXiv [here][paper], which was inspired by the free adversarial training paper [here][freepaper].*

[paper]: http://arxiv.org/abs/TODO
[freepaper]: https://arxiv.org/abs/1904.12843

## News
+ 12/19/2019 - Accepted to ICLR 2020
+ 12/23/2019 - arXiv posted and repository release

## What is in this repository? 
+ An implementation of the FGSM adversarial training method with randomized initialization for MNIST, CIFAR10, and ImageNet
+ [Cyclic learning rates](https://arxiv.org/abs/1506.01186) and mixed precision training using the [apex](https://nvidia.github.io/apex/) library to achieve DAWNBench-like speedups 
+ Pre-trained models using this code base
+ The ImageNet code is mostly forked from the [free adversarial training repository](https://github.com/mahyarnajibi/FreeAdversarialTraining), with the corresponding modifications for fast FGSM adversarial training

## Installation and usage
+ All examples can be run without mixed-precision with PyTorch v1.0 or higher
+ To use mixed-precision training, follow the apex installation instructions [here](https://github.com/NVIDIA/apex#quick-start)

## But wait, I thought FGSM training didn't work!
As one of the earliest methods for generating adversarial examples, the Fast Gradient Sign Method (FGSM) is also known to be one of the weakest. It has largely been replaced by the PGD-based attacked, and it's use as an attack has become highly discouraged when [evaluating adversarial robustness](https://arxiv.org/abs/1902.06705). Afterall, early attempts at using FGSM adversarial training (including variants of randomized FGSM) were unsuccessful, and this was largely attributed to the weakness of the attack. 

However, we discovered that a fairly minor modification to the random initialization for FGSM adversarial training allows it to perform as well as the much more expensive PGD adversarial training. This was quite surprising to us, and suggests that one does not need very strong adversaries to learn robust models! As a result, we pushed the FGSM adversarial training to the limit, and found that by incorporating various techniques for fast training used in the [DAWNBench](https://dawn.cs.stanford.edu/benchmark/) competition, we could learn robust architectures an order of magnitude faster than before, while achieving the same degrees of robustness. 

|          | CIFAR10 Acc | CIFAR10 Adv Acc (eps=0.1) | MNIST Acc | MNIST Adv Acc (eps=1.0) |
| --------:| ----------:|----------:| ---------:| ------------:|
| Standard     |       95% |       3% |     99% |          4% |
| l-inf robust |       66% |      61% |     98% |         48% |
| Adv training |       81% |      76% |     97% |         86% |
| Binarization |         - |        - |     99% |         14% |

## But I've tried this before, and it didn't work! 
In our experiments, we discovered several failure modes which would cause FGSM adversarial training to ``catastrophically fail''. If FGSM adversarial training hasn't worked for you in the past, then it may be because of one of the following reasons (which we present as a non-exhaustive list of ways to fail): 
+ FGSM step size is too large, forcing the adversarial examples to cluster near the boundary
+ Random initialization only covers a smaller subset of the threat model
+ Long training with many epochs and fine tuning with very small learning rates
All of these pitfalls can be avoided by simply using early stopping based on a subset of the training data to evaluate the robust accuracy with respect to PGD, as the failure mode for FGSM adversarial training occurs quite rapidly (going to 0% robust accuracy within the span of a couple epochs)