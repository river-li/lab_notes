---
title: POIROT
author: jeevansio
date: 2019-10-16 09:41:47
mathjax: true
categories: 
- Paper
tags:
- POIROT
- malware
---

## 概要 | ABSTRACT

POIROT 使用内核审计日志 (Kernel Audit Log) 为数据源构建起源图 (Provenance Graph, G<sub>p</sub>)，通过与用威胁情报构建的查询图 (Query Graph, G<sub>q</sub>) 进行匹配来检测系统是否存在威胁。作者使用了三种不同类型的数据集，三种不同的操作系统对 POIROT 的性能进行评估。评估结果表明，POIROT 能在几分钟之内，从百万数量级的节点中发现闭关指出攻击行为。

<!--more-->

## 简介| INTRODUCTION

- 威胁追踪存在的挑战
  - 查找规模庞大
  - 如何准确探测威胁和复盘攻击过程
  - 如何高效匹配
- POIROT
  - 使用单独的存储设备和日志服务器来保证内核审计日志的完整性
  - 使用 Graph Pattern Matching(GPM) 算法，根据 IOCs 之间的依赖关系进行匹配
  - 根据攻击成本/攻击者能力来选择攻击流程

## 相关研究 | RELATED WORK

- Log-based Attack Analytics
  - Opera et al.: DNS/web proxy log
  - Disclosure: 使用NetFlow logs检测C&C服务器
  - Hercule: 社区发现算法(Community Detection)重构攻击过程
- Provenance Graph Exploration
  - 使用内核审计日志(kernel audit log)构建起源图(provenance graph)
  - 分析方法
    - 动态二进制分析(dynamic binary analysis)
    - 源码注释(source code annotation)
    - 建模推理(modelling-based inference)
    - 记录回放(record-and-play)
    - 并行执行(parallel execution)
  - 分析来源
    - 警报类别(alert triage)
    - 攻击路径识别(0-day attack path identification)
    - 攻击分析重构(attack detection & reconstruction)
- Query Processing Systems
  - SAQL: 从系统活动中识别异常行为
  - AIQL: 从历史审计日志中分析攻击
  - CamQuery: 实时分析，重点在预防数据丢失、攻击检测、违规检测
  - Shu et al.: 使用工具进行人工辅助分析
- Behavior Discovery
  - Christodorescu et al.: 动态追踪
  - Kolbitsch at al.: 根据恶意样本的执行过程生成行为模型、
  - TGminer: 从审计日志中查找明显不同的行为模式
- Graph Pattern Matching
  - Fan et al.: 在多项式时间里，只能用
  - NeMa: 邻近子图匹配
  - G-Ray, Mage: 基于随机游走(Random-walk)，但不能分析包含混淆的攻击行为


## 方法概述 | APPROACH OVERVIEW

{% asset_img 4fde5323.png "POIROT Approach Overview" %}

### 构建起源图 | Provenance Graph Construction

- 结点：文件/进程等
- 边：信息流/因果关系

### 构建查询图 | Query Graph Construction

- 从威胁情报中提取IOC(OpenIOC, STIX, MISP, etc.)、行为模式
- 结点和边可添加name/type

{% asset_img 4825ecba.png "DeputyDog查询图" %}

### 图的匹配 | Graph Alignment

- GPM(Graph Pattern Matching)存在问题
  - 无法检测带属性的有向图
  - 无法处理大量结点
  - 穷举查找
- 匹配流程
  - 找出所有匹配的结点对
  - 从相似度最高的结点开始扩展，沿着与其类似的攻击流程查找结点
  - 使用影响因子(influence score)判断攻击者的攻击能力，排除无关或低相关性路径，缩小查找规模
  - 根据攻击影响力、攻击路径长度(length)和攻击成本(cost)评分，为攻击流程(flow)设置设置优先级
  - 对匹配到的图进行评分，高于阈值将发出预警并生成报告，否则继续寻找新的匹配

## 算法 | ALGORITHMS

### Alignment Metric

- Influence Score
  C<sub>min</sub>(i→j)：结点 i 到 j 的执行流程中的 ancestor 数量最小值，值越小表示越有可能是攻击流程。大多数攻击的都源于一个或一组 entry point（eg. 钓鱼邮件）
  $$
  \Gamma_{i,j}=\left\{
  \begin{array}{lcl}
  \max \limits_{i→j} \frac {1} {C_{min} (i→j)} \qquad & & \exists i→j | C_{min}(i→j) \leq C_{thr} \\
  0 & & otherwise
  \end{array}
  \right.
  $$

- Alignment Score
  计算所有执行流程的 influence score 的均值
  $$
  S(G_q::G_p) = \frac {1}{|F(G_q)|} \sum\limits_{(i→j)\in F(G_q)} {\Gamma_{k,l}|i:k \& j:l }
  $$

- Threshold
  alignment score 超过阈值 τ 将自动发出预警
  $$
  S(G_q::G_p) \geq \tau \\
    \tau = \frac{1}{C_{thr}}
  $$

### Best-Effort Similarity Search

- 查找所有候选结点(Find all Candidate Node Alignment)
  - 根据结点属性初筛，不考虑流程/路径
- 选择起始结点(Select Seed Nodes)
  - 攻击行为通常是从一个小的切入点出发，正常活动一般是重复进行的。所以此处首先应选择匹配数最少的结点作为起始结点
- 扩大搜索范围(Expand the Search)
  - 从起始节点开始往下级/上级结点开始，沿着/逆着路径方向遍历其它可达结点，influence score 等于或接近 0 时停止遍历，以提升遍历性能。
- 选择匹配的图(Graph Alignment Selection)
  - 从起始节点开始出发，在G<sub>p</sub>中选择能使 alignment score 最大化的节点，并建立连接
  - 贡献度计算
    $$
    A(i:k) = \sum\limits_{j:(i→j)\in F(G_q)}(1_{\{j:l\}}×\Gamma_{k,l}+(1-1_{\{j:l\}})× \max\limits_{m\in candidates(j)}(\Gamma_{k,m})) \\
    +\sum\limits_{j:(j→i)\in F(G_q)}(1_{\{j:l\}}×\Gamma_{l,k}+(1-1_{\{j:l\}})× \max\limits_{m\in candidates(j)}(\Gamma_{m,k}))
    $$
    - j, l 两点建立连接，则1<sub>{j:l}</sub>的值为 1，未建立连接时为 0
    - 第一项表示从节点 i 流出时的 influence score，第二项表示流入节点 i 时的 influence score。
    - 若节点已连接，则不继计算其他连接的 influence score
  - 选出贡献度最大的节点建立连接
    $$
    \mathop{arg\ max} \limits_{k\in K}A(i:k)
    $$

  
## 评估 | EVALUATION

预设 C<sub>thr</sub>=3（会影响 false postitve/negative）

### Exp 01: DARPA TC Dataset

- DARPA Transparent Computing program red-team vs. blue team adversarial engagement（模拟企业网络）
- 环境：Windows, BSD, Linux（启用内核审计报告）
  - BSD 1-4：FreeBSD_11.0_x64 + 带后门的Nginx
  - Win 1-2：Windows_7_Pro_x64 + 带有恶意宏的Excel文件 / 有                                                                            漏洞的Firefox
  - Linux 1-4：Ubuntu_12.04_x64 / Ubuntu_14.04_x64 + 浏览器的内存漏洞 / 恶意浏览器扩展
- 结果

{% asset_img 1571129853625.png %}

  - 第一轮循环时，所有环境中的 alignment score 均超过阈值，20轮重复后，7台机器的 alignment score 最大值仍然是第一轮的结果，其余的在第4/5轮得到最大值。
  


### Exp 02: Public Attacks

- 在隔离的环境中运行恶意样本，收集内核审计日志（模拟真实环境下的攻击）
- 变种检测评估
  - njRAT、DustySky：与威胁情报中的样本不同
  - Carbanak：从威胁情报中的109个样本中随机抽取
  - HawkEye：其他来源
- 与现成工具比对
  - RedLine/Loki/Splunk：基于独立的 IOC 检测，没有检测 IOCs 之间的依赖性和关联性
  - POIROT：基于 IOCs 之间的信息流和因果关系检测
- 检测结果

{% asset_img 1571189985348.png %}

  - B, F, H, P, R for **B**ehavior, **F**ile name, **H**ash value, **P**rocess name, **R**egistry
  - S, PE for alarms from Windows Security Mitigation and **PE**-Sieve
  - 所有的检测中 POIROT 的结果都超过了阈值 1/3，
  - POIROT 匹配到了 DustySky 的完整攻击流程，其余之匹配到了部分
  - Splunk 检测到当 Windows Security Mitigation 禁止 svchost 生成动态代码时，与 Carbanak 相关的 ETW 事件 
  - Loki 的 PE-Sieve 组件检测到部分样本尝试进行代码植入
- 结论

{% asset_img 1571154056571.png %}

  - 当使用了很多 IOCs 时，检测结果优于 POIROT
  - 攻击者的攻击模式不变时，POIROT 能更好地检测到攻击行为
  


### Exp 03: Benign Datasets

- 无攻击行为的绿色环境
  - 从威胁情报中提取了不同的攻击模式，从不同的起始节点开始进行了20次重复实验，选择最高的 alignment score 。结果显示，在计算 influence score 时，POIROT跳过掉了许多执行流程(flow)，最终的 alignment score 最高只有 0.16，远低于阈值。
- 选择合适的阈值

  {% asset_img 1571189928018.png %}

  - 当阈值设定在 [0.17, 0.54] 时，F-score 取得最大值。在此区间上去取中间值如 0.35 时，能最大化区分恶意攻击行为和正常行为。
  


### 高效性

- 横向比较
  
  {% asset_img 1571189835409.png %}

  - RedLine、Loki：离线工具，运行时间基于进程数和磁盘空间
  - POIROT、Splunk：在线查找，运行时间基于收集系统运行日志的时间、日志大小和系统活动数，实验中可在一分钟内检测出结果
    - Apache benchmark：衡量服务器响应时间 (web server responsiveness)
    - JetStream：衡量浏览器运行时间 (browser execution times)
    - HDTune：衡量磁盘工作时间 (heavy hard drive transactions)

- POIROT 性能（8-core 2.5GHz CPU, 150GB RAM）

  {% asset_img 1571189894007.png %}

  - 审计日志消耗
    - 比较磁盘（第2列）和内存（第4列）消耗可得，日志的平均压缩率在25%左右
  - 图的分析
    - 最后一列的时间表示图分析所消耗的时间，从提交G<sub>q</sub>开始计时，alignment score 达到阈值后停止计时
    - POIROT 的性能瓶颈
      - 扩大搜索范围时的时间消耗
      - G<sub>q</sub>节点名称和图的形状对性能也有影响，候选连接节点越多，运行时间也会相应增加



