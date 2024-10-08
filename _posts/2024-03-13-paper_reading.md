---
title: paper reading
tags:
    - paper
---

## CESM-HR

结果
- 1024 MPI 5.4x
- 18300 MPI 3.8x

方法
- 循环融合、函数内联、循环合并、data tiling、寄存器通信
- MPMD
- ocean block对齐

POP2优化部分
- 划分space-filling curve网格负载均衡
- athread
- MPI通信优化，PCSI solver优化

同门的另一篇文章介绍
方法

- data tiling
- DMA
- data reversing for vectorization
- CPE grouping，sub-kernel垂直层划分
- 双缓冲
- 内联函数
- if out of loop

## swNEMO

四级并行框架，列维度计算依赖
a highly adaptive, efficient four-level parallelization framework for OGCMs is proposed to release a new level of parallelism along the compute-dependency column dimension

众核RMA优化，动态cache调度策略，时间空间局部性，DMA带宽达到90理想带宽A many-core optimization method using blocking by remote memory access (RMA) and a dynamic cache scheduling strategy is applied, effectively utilizing the temporal and spatial locality of data. The test shows that the actual direct memory access (DMA) bandwidth is greater than 90 % of the ideal bandwidth after optimization, and the maximum is up to 95 %

混精，半精单精双精A mixed-precision optimization method with half, single and double precision is explored, which can effectively improve the computation performance while maintaining the simulated accuracy of OGCMs


通信瓶颈，大规模并行效率仍低于50，目前二维并行，三维并行中集成垂直仍挑战

- 四级三维空间并行算法
- 细粒度数据复用
- 复合分块算法和基于LDM cache的动态调度算法
- 引入半精


C-grid计算负载均衡分配，四级并行
- MPE 二维并行
- MPE-CPE异步并行 NEMO的正压求解器没有全局通信，仅halo通信，且显式方法中，边界信息交换与进程中的数据无关，可异步通信-计算
- CPE RMA行列通信 实现纬度深度分解，在深度维度上实现并行化
- 在经度维度向量化

stencil和时间依赖
- 利用RMA解决x指针问题，例如需三点和五点stencil
- 针对不同特性的kernel设计不同分块算法
    - z轴时间依赖 - 将数据沿y轴分解分配给CPE，x轴方向连续，（猜测直接分块，无RMA）
    - y轴时间依赖 - z轴分解，再将y轴分解为适合大小m，一次计算m*x的块，将同一z层的块逐个调度到同一个CPE上。（有参考意义，ldm_k方法可参考，减少LDM占用）
    - 非时间依赖 - 沿z轴和y轴分解，x方向上每个块一次性复制到LDM中，减少冗余halo，与y轴时间依赖类似，该项指多CPE可以完成一个z层，上述指单CPE时间上完成一个z层，存在halo时LDM占用更多，所以需处理
- 为提高RMA带宽利用率，打包数据，进行RMA通信和计算的重叠
- DDR4带宽利用率80，大部分kernel平均加速比可达40倍

LDCache动态调度
- 为了保证数据与内存一致性，定期更新存储
- CPE将数据打包发送到MPE的指定缓冲区，MPE轮询更新
- DDR4带宽利用率88.7，加速可达88倍

结果
从核12倍，主从异步65倍?
