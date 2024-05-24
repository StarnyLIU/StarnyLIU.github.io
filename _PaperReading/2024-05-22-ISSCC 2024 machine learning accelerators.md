---
title: 'Towards efficient generative AI and beyond-AI computing: New trends on ISSCC 2024 machine learning accelerators'
date: 2024-05-22
permalink: /PaperReading/2024/05/22/
tags:
  - 
  - 
---

## Four trends toward efficient generative AI and beyond-AI computing

# Trend 1: ML chips for generative AI
Related Work
* AMD MI300 series ()[]</br>
  * Technical composition: chiplet + AMD’s first-ever integration of data center class CPU, GPU + AMD Infinity Cache + 8-stack HBM3 memory system + advanced packaging technology
  
  * MI300X: CPU hosted PCIe® device
  针对传统的双路（2P）CPU服务器，托管8个GPU加速器，用于大型AI模型训练与推理
    * 8 AMD CDNA architecture-based GPU chiplets + a unified AMD Infinity Cache(用于保持多个chiplet之间的硬件一致性) + AMD Infinity FabricTM network on chip (NoC) + HBM3 memory + external Infinity Fabric and PCIe® Gen 5 links
  

  * MI300A: self-hosted accelerated processing unit (APU)
    * 3 AMD “Zen 4” CPU chiplets with 6 AMD CDNA 3 GPU chiplets + AMD Infinity Fabric NoC + sharing a unified AMD Infinity Cache backed by HBM3 DRAM
  * The most impressive part is that MI300 makes 13 pieces of silicon behave as one chip (3D integration, chiplet, packing, hybrid bonding .etc). 参见以下[IEEE Spectrum](https://spectrum.ieee.org/amd-mi300)
  * About [3D V-Cache](https://ieeexplore.ieee.org/document/9731565)
  * About [hybrid bonding](https://www.sciencedirect.com/science/article/pii/S0026271424000520) and this [zhihu blog](https://zhuanlan.zhihu.com/p/656053590), this [blog](https://www.ab-sm.com/a/22679)
  * AMD-Chiplet  [zhihu blog](https://zhuanlan.zhihu.com/p/658391280)
  Paper:[Pioneering Chiplet Technology and Design for the AMD EPYC™ and Ryzen™ Processor Families : Industrial Product](https://ieeexplore.ieee.org/document/9499852) ISCA 2021
    * 理解 The Product Portfolio Multiplier
    * 摩尔定律失效导致的问题：半导体公司需要继续交付晶体管数量更多的产品，以市场友好的价格为客户提供更多的功能和计算能力，但摩尔定律的瓦解推迟了交付额外器件的新工艺节点的可用性，成本也在增加。
    * Figure 5 单片SoC传统制造流程，需要通过 "known good die" (KGD) testing；使用chiplet使得被丢弃的硅数量更少，还可以测试单个chiplet的最大性能（clock），并将更快的chiplet组装成fast cores； ———— chiplet技术提高原始功能良率(raw functional yield rates)并增加高性能产品的供应
    * 在引入新工艺技术节点的早期，良率往往低于成熟工艺。因此，试图在新新工艺节点早期构建大型SoC更具经济挑战性。然而，通过利用一组产量高得多的小芯片，可以比传统的单片设计更早地过渡到新的工艺节点。
    * 通过组装chiplet得到超越光刻极限的单片芯片面积
    * chiplet的代价：
      * 更多的前期工作，将SoC划分为正确数量和正确类型的芯片（SoC分区要满足成本限制、性能要求、IP易用性和硅重用等要求）
      * 还需要新的芯片间通信路径。与片上金属相比，这些互连涉及更长的路由，可能具有更高的阻抗、更低的可用带宽、更高的功耗和/或更高的延迟。互连开销还可能包括用于交叉电压和时序域、协议转换和/或串行-反序列化器(serializer-deserializers， SerDes)的电路，这些电路都代表了在单片设计中不存在的额外功率和硅面积开销
      * 其他电路和功能也需要以每个芯片为基础进行复制。e.g.测试和调试接口(用于组装前测试单个芯片)，时钟生成和分配，电源管理，片上温度传感器，I/O(例如USB, SATA)。因此，单个单片T型晶体管SoC的总硅面积面积(SoC(T))通常小于n个小芯片的总面积，其中每个小芯片具有面积(SoC(T/n)+K)的芯片面积，其中K代表上面讨论的额外开销(例如，芯片间接口，测试电路)。然而，与单片方法相比，为n个独立的小芯片使用更多的总硅面积仍可能导致更低的总成本
    * 第一代AMD EPYC™处理器（霄龙服务器处理器）：32 cores, 8 DDR4, 128 PCIe®gen3, 在14nm制程中需要777mm2的芯片面积（假设）
      * 采用四个芯片组合的方式，每个芯片提供8个“Zen”CPU内核，2个DDR4内存通道，32个PCIe通道每个晶片的14nm制程的晶片面积为213 mm2，总的晶片面积为4 * 213mm= 852 mm2。与假设的单片32核芯片相比，~10%的芯片面积开销，但四晶片设计的最终成本仅为单片方法的约0.59x
      * 优势
        * 1、 即使面对一些制造缺陷，也可以利用每块晶圆中芯片总数的更高比例；
        * 2、只需一次chiplet设计(即一个掩模组，一次流片)，就可以提供多个不同的产品，而这些产品传统上需要多个独立的不同SOC
        * 3、 Chiplet方法使得提供具有完整内存和I/O功能的产品更加实用 ———— 专用的16核单片SoC本身无法承受如此多内存通道和I/O通道所需的额外芯片面积的成本负担。然而，单个小芯片的较低成本加上在多个产品上摊销额外内存和I/O接口的能力，可以使这成为一种经济可行的方法，同时实现有价值的产品差异化，以满足客户需求。
      * 劣势
        * 1、chiplets need to communicate across the Infinity Fabric™ on-package (IFOP) interconnect, which are implemented as point-to-point links directly on the organic package substrate. --->  additional interconnect latency
        * 2、与PCIe gen3等封装外I/O的SerDes(每比特消耗约11pJ)相比，IFOP SerDes针对更短的封装基板布线长度进行了精心优化（每比特2pJ的能效），但与单片芯片（片上互连功率通常远低于每比特1pJ）相比还有功耗开销
        * 3、一个chiplet上有8个CPU core，总共 8个DDR4内存通道中只有2个在同一个chiplet上。这意味着，在没有非一致性内存访问（non-uniform memory access, NUMA）数据管理和线程固定的情况下，某些内存请求必须由“远程”内存通道提供服务
        * 4、对产生DRAM page misses的请求，同一chiplet上的本地内存通道的内存请求典型延时为90ns；访问不同chiplet、同一个socket的远程内存通道的延迟为141ns。这里额外的延时是 IFOP links和通过每个chiplet的本地data fabric（也称为片上网络或 NoC）的additional hops（布局设计中添加的额外连线或信号传输路径）的往返组合造成的。所以对于在8个内存通道之间均匀交错的内存访问模式，平均内存延迟为128ns。这些socket内NUMA影响是第二代 AMD EPYC™ 处理器解决的问题之一。
    * 第二代AMD EPYC™处理器：双芯片方法。7nm 核心复合芯片(CCD)~74mm2 + 12nm I/O芯片(IOD)~416mm2（8个DDR4存储通道，128个PCIe gen4 I/O通道）
      * "Zeppelin" chiplet 对工艺、面积与成本进行估算与权衡
      * 1个IOD与8个CCD组装在一起。每个CCD提供8个“Zen 2”CPU内核，一个socket中可以集成64个内核。

  

  </br>
* THU first heterogenous CIM-based accelerator for image-generative diffusion models