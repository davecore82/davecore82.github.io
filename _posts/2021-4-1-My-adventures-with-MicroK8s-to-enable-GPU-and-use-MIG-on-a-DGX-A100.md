---
layout: post
title: My adventures with MicroK8s to enable GPU and use MIG on a DGX A100
---

I recently had the chance to play with an Nvidia DGX A100 server. The DGX A100 is a beast with 8x NVIDIA A100 Tensor Core GPUs and a total of 320 GB of GPU Memory. 

There's a new Nvidia feature called MIG (Multi Instance GPU) that lets you split one GPU into up to 7 slices so that more containers/pods can use GPUs simultaneously. I ended up using MicroK8s and Ubuntu 20.04 to do this in Kubernetes. It wasn't the easiest thing to do so I posted my findings on the [Kubernetes discuss forum](https://discuss.kubernetes.io/t/my-adventures-with-microk8s-to-enable-gpu-and-use-mig-on-a-dgx-a100/15366) so that others could benefit from my experience.

In summary, the steps were the following:

 1. Make sure to completely remove Nvidia drivers from the host
 2. Blocklist the nouveau driver 
 3. Install fabric manager
 4. Install MicroK8s 1.21/beta 
 5. Enable DNS and make sure it works 
 6. Enable GPU and fabric manager 
 7. Enable MIG
 8. Create the GPU slices

Check out my post on the Kubernetes discuss forum for more details:
[https://discuss.kubernetes.io/t/my-adventures-with-microk8s-to-enable-gpu-and-use-mig-on-a-dgx-a100/15366](https://discuss.kubernetes.io/t/my-adventures-with-microk8s-to-enable-gpu-and-use-mig-on-a-dgx-a100/15366)

