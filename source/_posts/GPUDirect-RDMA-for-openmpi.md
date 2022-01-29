---
uuid: c7e8105f-1b87-428f-c7db-c7f8905e8c69
title: GPUDirect RDMA for openmpi
tags:
  - Linux
  - cuda
categories:
  - 技术
toc: false
date: 2020-04-10 21:41:37
---

#### 背景
对于MPI跨节点项目，对GPU上的数据一般需要先cudaMemcopy到Host，再通过mpi_send出去，另一设备通过mpi_Recv到Host内存，再cudamemcopy到GPU显存，这一过程明显要费时。

## 1. compile openmpi with --with-cuda
这一编译方法可以让openmpi对显存的数据操作，但是它只是减少了代码的书写量，从GPU到Host的数据传输在背后任然在执行。时间甚至比手写cudaMemcopy还要长。为了解决这一问题，就采用了GPUDirect RDMA 技术。
[openmpi run with cuda](https://www.open-mpi.org/faq/?category=runcuda)
[Building CUDA-aware Open MPI](https://www.open-mpi.org/faq/?category=buildcuda)
Open MPI v1.7.4 and later have added some support to take advantage of GPUDirect RDMA on Mellanox cards. All the details about Mellanox hardware as well as software needed to get things to work can be found at the Mellanox web site. Note that to get GPUDirect RDMA support, you also need to configure your Open MPI library with CUDA 6.0.
可以看到要想使用GPUDirect RDMA，任然需要对openmpi 加上cuda参数编译。

## 2. 查看GPUDirect RDMA 信息
1. To see if you have GPUDirect RDMA compiled into your library, you can check like this:
```bash
shell$ ompi_info --all | grep btl_openib_have_cuda_gdr
   MCA btl: informational "btl_openib_have_cuda_gdr" (current value: "true", data source: default, level: 4 tuner/basic, type: bool)
```
2. To see if your OFED stack has GPUDirect RDMA support, you can check like this:
```bash
shell$ ompi_info --all | grep btl_openib_have_driver_gdr
   MCA btl: informational "btl_openib_have_driver_gdr" (current value: "true", data source: default, level: 4 tuner/basic, type: bool)
```
3. To run with GPUDirect RDMA support, you have to enable it as it is off by default:
```bash
mpirun --mca btl_openib_want_cuda_gdr 1 ...
```
官网的主要介绍就是这样，但是在自己设备上测试就没这么容易了，问题出现在btl_openib_have_driver_gdr这一步，结果如下：
![gdr.PNG](/images/2020/04/10/e8a1c790-7b2e-11ea-acfc-35c1710d51fd.PNG)它显示的值是false，初步判断是缺少驱动所致，问题解决过程如下：
## 3. 安装 nv_peer_mem
安装过程参考浪潮公司的专利内容：
[一种基于GPUDerict RDMA测试方法](https://patentimages.storage.googleapis.com/c7/56/c9/9bba52318b9458/CN105550085A.pdf)
具体步骤如下图所示：
![nv.PNG](/images/2020/04/10/afa1d740-7b2f-11ea-acfc-35c1710d51fd.PNG)

运行lsmod |grep nv_peer_mem检查是否安装成功
```bash
lsmod | grep nv_peer_mem
```

安装后再查看btl_openib_have_driver_gdr信息就正常了
![gdr2.PNG](/images/2020/04/10/e176a570-7b2f-11ea-acfc-35c1710d51fd.PNG)
这时我通过测试节点间的显存数据mpi传输，发现速度有明显提升，其中GPU-to-GPU的速度跟从Host到Host速度相差无几，测得GPU-to-GPU速度超过8GB/s，与节点内的速度处在相同数量级。由于避免了在Host内存的中转，速度提高至少50%。

*其中的原理是：*
直接访问GPU内存，避免访问固定(pinned) CUDA主机内存时不必要的系统内存拷贝和CPU的开销，加速了与网络和存储设备之间的通信可以在同一系统中的一个GPU直接访问另一个GPU使用直接的高速DMA传输，增加了 P2P的内存访问，真正释放了主机CPU资源，消除主机了CPU中不必要的频繁数据传输，完全不参与输入的RDMA操作；包括HCA卡、GI3U卡、GI3U必备的Nvidia Driver^Nvidia CUDA toolkit，及infiniband必备的MLNX_0FED驱动外，以及一个GPU与IB卡通信的nv_peer_mem包。