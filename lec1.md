# HPML Lecture 1 — Course Intro / Defining Terms / Historical Perspective

> Lecture 1 偏概览和历史，"硬考点"不算多，但**术语定义、年代分期、关键数字、需要 Efficient AI 的三大理由**很容易出选择/简答/判断题。

---

## 1. 这份 slides 最重要的 5–10 个考点

1. **HPC 的三大组成维度**：Computing speed (FLOPS) / Storage (capacity, transfer rate, technology) / Computer-to-computer communication
2. **AI vs ML 的区分**：ML 是 "用数据优化算法"，AI 范围更大且定义随时间变化（"frontier of computer capabilities"）
3. **Performance Optimization 三步法**：**Measure → Analyze → Optimize**（循环迭代）
4. **计算机历史五个时代分期**及代表机器、技术、年份边界
5. **并行架构概念**：MIMD vs SIMD；网络拓扑 crossbar / mesh / torus
6. **HPC 与 AI 的共同点**：都关注当前能力的"frontier"
7. **Need for Efficient & Sustainable AI 三大理由**：① Model Complexity ② Computational Demands ③ Carbon Footprint
8. **关键缩放数字**：Transformer 模型 **240×/2yrs**；训练算力 **doubling every 3.5 months**；HW memory 仅 **2×/2yrs**（→ Memory Wall）
9. **Foundation Model 定义**：大规模 + 自监督 + 无标注数据 + 可适配下游任务
10. **GPT-3 标志性数字**：175B 参数、285k CPU + 10k GPU、4–12M$ 训练成本、~552 吨 CO₂

---

## 2. 核心知识点精炼总结

### 2.1 High-Performance Computing 的衡量维度
- **Computing speed**：主要用 **FLOPS**（Floating-point Operations Per Second）；早期机器没浮点硬件，只能用 MIPS / ops per sec
- **Storage**：三个子维度 → Technology（paper → magnetic → SSD）/ Capacity（bytes → TB）/ Transfer rate（bits/sec）
- **Communication**：computer-to-computer，1950s 之前几乎不存在；含长距离通信 + 超算内部 interconnect
- **重点**：HPC 不是单一指标，而是 **算力 + 存储 + 通信** 三条曲线一起增长

### 2.2 AI vs ML
- **AI**：定义随时间变化，永远指"当前计算机能力的前沿任务"
- **ML**：用 example data 优化算法的子集
- **非 ML 的 AI 例子**（常被问）：rule-based 专家系统、符号逻辑、ELIZA、Cyc 这种 commonsense reasoning、LISP 程序

### 2.3 Performance Optimization 方法论
- **三步循环**：Measure（测） → Analyze（分析瓶颈） → Optimize（优化）
- **核心思想**：System-level approach；优化要先有 measurement，不能凭感觉
- **可获得收益**：单点优化可达 100× 加速
- **应用方向**：① 解决问题更快 ② 解决更大的问题

### 2.4 历史分期（必背表）

| 时代 | 年份 | 代表技术 | 代表机器 / 事件 |
|---|---|---|---|
| Pre-electronic | – 1946 | 机械 | Babbage Analytical Engine（~1 FLOPS）；Hollerith 打孔卡（1890 census, 36 bytes/card） |
| Early electronic | 1946 – 1958 | 真空管 + 磁芯 | **ENIAC (1946, 500 FLOPS)**；NORC (1954) |
| Transistor / mainframe | 1958 – 1971 | 晶体管 | IBM 7030 Stretch、CDC 6600 (1964)、IBM 1301 磁盘、ARPANET (packet switching) |
| Integrated circuits / PC | 1971 – 1999 | IC、PC | **Cray-1 (1976)**、Cray X-MP/Y-MP；Intel ASCI Red 突破 1 TFLOPS (1997) |
| Internet / GPU | 1999 – 现在 | 大规模并行 + GPU | Blue Gene、Roadrunner (1 PFLOPS, 2008)、Frontier (1 EFLOPS, 2021) |

**记忆技巧**：四个分界点 = 1946（ENIAC）/ 1958（晶体管）/ 1971（IC）/ 1999（GPU+互联网）

### 2.5 并行架构概念
- **MIMD**：Multiple Instruction, Multiple Data — 不同处理器跑不同指令、不同数据（最通用）
- **SIMD**：Single Instruction, Multiple Data — 同一指令应用到多组数据（GPU、向量处理器的本质）
- **拓扑结构**（处理器互联方式）：
  - **Crossbar**：任意两点直连，带宽高但成本 O(N²)
  - **Mesh**：网格状，简单可扩展
  - **Torus**：mesh 边界回连成环，缩短最远距离（Blue Gene、TPU pod 用）

### 2.6 网络性能（GPU 集群必懂）

| 网络 | Latency (μs) | Bandwidth (Gb/s) |
|---|---|---|
| 10 GigE | 10 | 4 |
| 40 GigE | 40 | 4 |
| InfiniBand EDR | 100 | 1 |
| **NVLink** | **>400** | **0.1–0.2** |

> ⚠️ 注意：slides 这个表的 latency 单位看起来很奇怪（NVLink latency 不该比以太网高）——**考试若问"为什么大模型训练用 NVLink/IB 不用以太网"，答：低延迟 + 高带宽**。深度学习/分布式训练需要 high BW + low latency，普通以太网不够。

### 2.7 Foundation Models
- **定义**：大规模 AI 模型，在**海量无标签数据**上通过**自监督学习**训练，得到可**适配多个下游任务**的模型
- **代表**：Gemini、LLaMA、GPT-3/4、Claude、Granite、Falcon
- **范式**：Raw data → Foundation model → Efficient adaptation → Insights

### 2.8 Need for Efficient & Sustainable AI（**最核心考点之一**）

| # | 维度 | 核心数字 / 事实 |
|---|---|---|
| ① | **Increased Model Complexity** | 参数量约 **10×/year**；Transformer **240×/2yrs**，但 AI HW memory 仅 **2×/2yrs** → "AI Memory Wall" |
| ② | **Unbounded Computational Demands** | 训练算力 **doubling every 3.5 months**；远超 Moore's Law (2×/2yrs)；Transformer 训练算力 **750×/2yrs** |
| ③ | **Increasing Carbon Footprint** | 训练 1 个模型 ≈ **5 辆车终身排放**；NAS Transformer ~626k lbs CO₂；GPT-3 训练 ~552 吨 CO₂ ≈ 3 次 NY↔SF jet 往返 |

**结论金句**：*"Larger models need more hardware — business as usual is not sustainable."*

---

## 3. 关键数字 / 公式速记

- **FLOPS** = Floating-point Operations Per Second
- **Moore's Law**：2× / 2 yrs（CPU 晶体管/算力）
- **Transformer 模型规模**：240× / 2 yrs
- **AI 训练算力增长**：2× / **3.5 个月**
- **HW memory 增长**：2× / 2 yrs
- **GPT-3**：175B params, 285k CPU + 10k GPU, $4–12M, 310k ExaFLOP, 552 吨 CO₂
- **历史 FLOPS 跨度**：1（Babbage 1840s）→ 500（ENIAC 1946）→ 10⁹（Cray-2 1985）→ 10¹² (1997) → 10¹⁵ (2008) → 10¹⁸ (2021 Frontier)

---

## 4. 容易混淆的点

| 对比 | 区别 |
|---|---|
| **AI vs ML** | AI 包含规则系统/符号推理；ML 必须"从数据中学习" |
| **MIMD vs SIMD** | MIMD 指令各异（CPU 多核）；SIMD 同指令多数据（GPU、向量化） |
| **Crossbar vs Mesh vs Torus** | Crossbar = 全互联（贵）；Mesh = 网格（便宜，远点慢）；Torus = 闭环 mesh（远点也短） |
| **Compute vs Memory wall** | 算力 ↑ 比 memory ↑ 快得多 → memory 成新瓶颈 |
| **FLOPS vs MIPS** | FLOPS 含浮点；MIPS 只算整数指令；早期机器没 FP 硬件用 MIPS |
| **Storage 三参数** | Technology（用什么）≠ Capacity（多大）≠ Transfer rate（多快） |
| **Foundation model vs 普通 ML 模型** | FM 用**自监督 + 无标签 + 可适配多任务**；普通 ML 模型针对单一任务 |

---

## 5. 这份 slides 最可能出的题型

### 题型 1：年代/机器对应题
- **识别方式**：给一个机器名 / 年份 / 技术，问属于哪个时代
- **解题**：记住 4 个分界年份 1946 / 1958 / 1971 / 1999；记住代表机器 ENIAC / Cray-1 / Blue Gene

### 题型 2：术语定义/区分题
- **识别方式**：让你区分 AI/ML、MIMD/SIMD、HPC 三维度
- **解题**：用上面对比表里的"一句话区分"

### 题型 3：为什么需要 Efficient AI（简答）
- **解题模板**：分三点答 — Model complexity ↑、Compute demand ↑（每 3.5 个月翻倍）、Carbon footprint ↑（GPT-3/Transformer 数字举例）

### 题型 4：性能优化方法论
- **识别方式**：问"如何系统地优化一个 ML 程序的性能"
- **解题**：Measure → Analyze → Optimize 循环 + profiling 工具 + 找 bottleneck

### 题型 5：网络拓扑选择
- **识别方式**：给一个超算/集群场景问选什么 interconnect
- **解题**：DL 训练 → low latency + high BW → NVLink / InfiniBand，**不能用普通 Ethernet**

### 题型 6：缩放数字辨识
- **识别方式**：判断题如 "Transformer 模型规模每 3.5 个月翻倍"（错，是训练算力）
- **解题**：分清 — **模型规模 240×/2yr**，**训练算力 2×/3.5 月**，**HW memory 2×/2yr**

---

## 6. 一页速记版（考前最后看）

- HPC 三维度：**算力(FLOPS) + 存储(容量/速率/技术) + 通信**
- AI ⊃ ML；ML = 用数据优化的算法
- **Optimize 流程**：Measure → Analyze → Optimize（迭代）
- 五个时代：**机械 / 真空管 / 晶体管 / IC / GPU+网络**；分界点 **1946 / 1958 / 1971 / 1999**
- ENIAC=1946=500 FLOPS；Cray-1=1976=160 MFLOPS；Frontier=2021=1 EFLOPS
- 并行：**MIMD vs SIMD**；拓扑：**Crossbar / Mesh / Torus**
- DL 训练网络要 **低延迟 + 高带宽** → InfiniBand / NVLink
- **Foundation Model** = 大模型 + 自监督 + 无标签 + 多任务适配
- 三大需求：**模型 ↑（10×/yr） + 算力 ↑（2×/3.5月）+ 碳排放 ↑**
- **AI Memory Wall**：模型 240×/2yr ≫ HW memory 2×/2yr
- GPT-3：**175B / 285k CPU / 10k GPU / 4–12M$ / 552 吨 CO₂**

---

## 7. 自测题（附简短答案）

**Q1**. 列出 HPC 关注的三个维度，并各举一个衡量指标。
- 答：Computing speed（FLOPS / MIPS）、Storage（容量 bytes、传输速率 bits/s）、Communication（带宽、延迟）。

**Q2**. AI 和 ML 的关系是什么？给一个属于 AI 但不属于 ML 的例子。
- 答：ML ⊂ AI；ML 必须从数据中学习。非 ML 的 AI：专家系统（如 R1）、ELIZA、符号逻辑推理（Logic Theorist）、Cyc。

**Q3**. Performance optimization 的标准三步流程是什么？
- 答：Measure → Analyze → Optimize，循环迭代。

**Q4**. 为什么深度学习训练集群通常用 InfiniBand 或 NVLink 而不是普通 Ethernet？
- 答：DL 训练需要 high bandwidth + low latency 进行 gradient/参数同步，普通 Ethernet 带宽和延迟都不够。

**Q5**. 写出近年三个关键缩放速度，并说明它们暴露了什么问题。
- 答：Transformer 模型规模 240×/2yr；训练算力 2×/3.5 月；HW memory 仅 2×/2yr。模型增长远快于内存增长 → AI Memory Wall。

**Q6**. Foundation model 的三个本质特征是什么？
- 答：① 大规模 ② 用大量无标签数据自监督训练 ③ 可适配多个下游任务。

**Q7**. 简述 SIMD 和 MIMD 的区别，并说明 GPU 属于哪一类。
- 答：SIMD 同指令多数据；MIMD 不同指令不同数据。GPU 主体是 SIMD（更准确叫 SIMT）。

**Q8**. 列出 5 个计算机时代的边界年份及标志事件。
- 答：—1946（ENIAC）/ 1946–1958（真空管）/ 1958–1971（晶体管 → IC）/ 1971–1999（IC + PC）/ 1999– 至今（GPU + Internet）。

**Q9**. 为什么说 "business as usual is not sustainable"？给三个理由。
- 答：① 模型复杂度爆炸（10×/yr）② 训练算力需求每 3.5 月翻倍 ③ 碳排放和成本巨大（GPT-3 ≈ 552 吨 CO₂，4–12M$）。

**Q10**. Crossbar、Mesh、Torus 三种拓扑各自的特点？
- 答：Crossbar：任意两点直连，带宽最高但成本 O(N²)；Mesh：网格连接，简单便宜但远点延迟高；Torus：mesh 边界相连成环，缩短最远距离，常用于大规模超算。