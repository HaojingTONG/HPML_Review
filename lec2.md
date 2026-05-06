# HPML Lecture 2 — ML Performance Optimization

## 1. 这份 slides 最重要的 10 个考点

- **Amdahl's Law 公式及其含义**（识别 critical section，算 overall speedup 上限）
- **Roofline Model**（arithmetic intensity 计算 + 判断 memory-bound vs compute-bound）
- **Strong Scaling vs Weak Scaling** 的定义区别与实际场景
- **Profiling vs Tracing** 的原理差异与使用场景
- **Speedup / Scaling Efficiency 公式**
- **平均值的选择**：arithmetic / harmonic / geometric mean 各用在哪里
- **数据移动的延迟层次**（寄存器 → L1/L2/L3 → DRAM → PCIe → 网络）
- **时间的类型**：Real / CPU / User / System / Non-CPU 的定义与关系
- **Bottleneck 两大类**：Data Movement-bound vs Compute-bound 的判断
- **Peak FLOPS 的计算公式**及影响因素

---

## 2. 核心知识点精炼总结

### 2.1 Performance Optimization 三步法

- **Measure → Analyze → Optimize**（循环迭代）
- Measure：执行 workload，收集时间/吞吐量/profiling 数据
- Analyze：理解硬件/软件栈，找 Critical Path，找 Bottleneck
- Optimize：改代码或配置，对症下药

### 2.2 性能指标

- **Latency（延迟）**：单次操作的执行时间 $t$
- **Throughput（吞吐量）**：$\text{# operations} / t$
- **FLOPS**：$\text{# floating point ops} / t$
- **Speedup** of B w.r.t. A：$t_A / t_B$
- **Parallel Speedup**：$t_{\text{serial}} / t_{\text{parallel}}$

### 2.3 Scalability（可扩展性）

- **Scaling Efficiency**：$E = \dfrac{t_{\text{serial}}}{t_{\text{parallel}} \times p} \leq 1$，$p$ 是进程/线程数
- **Strong Scaling**：问题规模固定，增加 $p$ → 期望缩短时间。瓶颈：同步开销随 $p$ 增加
- **Weak Scaling**：问题规模随 $p$ 等比增加 → 期望时间不变。场景：fold bigger protein or more proteins in same time

**"Not Good" speedup 的原因**：随着 $p$ 增加，global sync 和 communication 占比越来越大，computation 占比下降，效率受限。

### 2.4 Computing Averages（平均值）

| 场景 | 用哪种平均 |
|---|---|
| 平均执行时间 | 算术均值 Arithmetic mean |
| 平均吞吐量（固定 $t$） | 算术均值 |
| 平均吞吐量（固定 #ops） | 调和均值 Harmonic mean |
| 平均 speedup / slowdown / 任意比值 | 几何均值 Geometric mean |

### 2.5 时间类型（Time Definitions）

| 类型 | 含义 |
|---|---|
| Real / Wall Clock | 实际经过的时间（包括 I/O 等待等） |
| CPU Time | 执行 CPU 指令的时间 = User + System |
| User Time | 用户空间代码时间 |
| System Time | 内核空间（OS）代码时间 |
| Non-CPU Time | I/O 等待、虚拟化等 idle 时间 |

**关键不等式**：`real >= user + sys`

- Linux：`time ./executable` → 毫秒精度
- C：`clock_gettime(CLOCK_MONOTONIC, ...)` → 纳秒精度
- Python：`time.monotonic()` / `time.monotonic_ns()`

### 2.6 Profiling vs Tracing

**Profiling**（采样为主）：
- **Sampling**：周期性 IRQ 中断 → 记录当前指令计数器 → 统计近似时间分布
  - 优点：overhead 低
  - 缺点：采样少时误差大（error ∝ 1/√N）
- **Counting**：精确计数事件（软件/硬件 performance counter），如 L2 cache miss、branch misprediction 次数

**Tracing**（插桩）：
- 在代码中显式插入追踪原语（`RECORD_TRACE_EVENT`）
- 精确记录每个事件时间戳，开销比 profiling 高
- 工具：`strace`（系统调用追踪）、`ftrace`（内核函数追踪）

**关键区分**：profiling 是统计推断（近似），tracing 是精确记录（全量）。

### 2.7 Amdahl's Law

$$S(p, s) = \frac{1}{(1 - p) + p/s}$$

| 变量 | 含义 |
|---|---|
| $S$ | 整个程序的加速比 |
| $p$ | 被优化代码占总时间的比例（$p$ 高 → 称为 critical section） |
| $s$ | 被优化部分的加速比 |

**核心结论**：当 $s \to \infty$，$S_{\max} = 1/(1-p)$。即使把某段代码优化到极限，整体加速比上限由 $p$ 决定。

### 2.8 Bottleneck 分析

**两大类 bottleneck**：
1. **Data Movement**（内存/网络带宽/延迟限制）
2. **Computation**（CPU/GPU FLOPS 限制）

**数据移动延迟层次（从快到慢）**：

| 层次 | 延迟（2.0 GHz CPU cycles） | 粒度 |
|---|---|---|
| Registers | 1 cyc | 4 bytes |
| L1 Cache | 3 cyc | 64 bytes |
| L2 Cache | 10 cyc | 64 bytes |
| L3 Cache | 75 cyc | 64 bytes |
| DRAM | 100 cyc | 64 bytes |
| IB Network | 2000+ cyc | 128B+ |
| GPU mem (PCIe) | 5000+ cyc | 64 bytes+ |
| Ethernet | 20000+ cyc | 64 bytes+ |

规律：**越靠近 CPU → 粒度小、延迟低、带宽高**

### 2.9 Roofline Performance Model

**核心思想**：实际 FLOPS 受两个上限约束——内存带宽（memory-bound）和 CPU/GPU 峰值算力（compute-bound）。

**关键公式**：

$$\text{Arithmetic Intensity (AI)} = \frac{\text{# arithmetic ops (FLOP)}}{\text{DRAM data (bytes)}}$$

$$\text{Attainable FLOPS} = \min(\text{Peak FLOPS},\ \text{Peak BW} \times \text{AI})$$

**crossover point**：$\text{AI}_{\text{cross}} = \dfrac{\text{Peak FLOPS}}{\text{Peak BW}}$
- AI < crossover → **memory-bound**（优化方向：减少 DRAM 访问，如 cache blocking）
- AI > crossover → **compute-bound**（优化方向：提高计算密度，如 vectorization）

**Peak FLOPS 公式**：

$$\text{Peak FLOPS} = \text{#cores} \times \text{frequency(cyc/s)} \times \text{FLOPS/cycle}$$

FLOPS/cycle 受 SIMD 指令（AVX 等）和精度（DP/SP/HP = 64/32/16 bit）影响。

### 2.10 性能优化技术

| 优化方向 | 技术 | 原理 |
|---|---|---|
| 减少延迟 | Cache Blocking（分块） | 数据分成 cache 大小的块，减少 DRAM 访问 → AI 上升 |
| 增加并行 | Vectorization / SIMD | 1 条指令执行多个 FLOP（2/4/8/16 倍）→ FLOPS 上升 |
| 增加并行 | Thread-level Parallelism | 多核并行 |
| 增加并行 | Multi-node Parallelism | 分布式计算 |

### 2.11 浮点误差（Floating Point）

- FP32：1 sign + 8 exp + 23 mantissa，bias=127
- FP64：1 sign + 11 exp + 52 mantissa，bias=1023
- **精度（relative error）** 由尾数位数决定
- **Catastrophic cancellation**：$C = B + A - A$ 当 $B \ll A$ 时，$C$ 相对误差 → 1

---

## 3. 公式 / 定律 / 算法步骤

### 公式汇总

| 公式 | 名称 | 变量说明 |
|---|---|---|
| $S(p,s) = \frac{1}{(1-p)+p/s}$ | Amdahl's Law | $p$=被优化部分比例，$s$=该部分加速比 |
| $E = \frac{t_{\text{serial}}}{t_{\text{parallel}} \times p}$ | Scaling Efficiency | $p$=进程数 |
| $\text{AI} = \frac{\text{FLOP}}{\text{DRAM bytes}}$ | Arithmetic Intensity | FLOP 是操作数量（非速率） |
| $\text{Peak FLOPS} = \text{cores} \times f \times \frac{\text{FLOP}}{\text{cycle}}$ | 峰值算力 | $f$=时钟频率 |
| $\text{Harmonic mean} = \frac{n}{\sum \frac{t_i}{\text{#ops}}}$ | 调和均值 | 固定 #ops 时的平均吞吐量 |
| $\text{Geo mean} = \left(\prod s_i\right)^{1/n}$ | 几何均值 | 平均 speedup |

### Roofline 解题步骤

1. 查 HW spec → Peak FLOPS 和 Peak BW
2. 计算 $\text{AI}_{\text{cross}} = \text{Peak FLOPS} / \text{Peak BW}$
3. 分析程序 → 计算 AI = FLOP / DRAM bytes
4. 比较：AI < cross → memory-bound；AI > cross → compute-bound
5. 计算 attainable FLOPS = min(Peak FLOPS, BW × AI)
6. 根据 bottleneck 选优化策略

---

## 4. 容易混淆的点

- **FLOP vs FLOPS**：FLOP 是浮点操作**数量**（单位：个），FLOPS 是**速率**（FLOP/s）。Roofline 的 AI 用的是 FLOP（量），不是速率。
- **Strong vs Weak Scaling**：Strong = 问题大小**固定**，加进程提速；Weak = 问题**随进程数等比增大**，时间保持不变。记忆：Strong = 同样的蛋白质折叠更快，Weak = 同样时间折叠更多/更大的蛋白质。
- **Profiling vs Tracing**：Profiling 是**统计采样**（低开销，近似），Tracing 是**显式插桩**（高开销，精确）。
- **Real time vs CPU time**：Real ≥ CPU time，差值 = I/O 等待（Non-CPU time）。多线程时 CPU time 可能 > Real time（多个核心的 CPU time 叠加）。
- **Arithmetic mean vs Harmonic mean**：计算平均**时间**用 arithmetic，计算平均**吞吐量**（固定工作量）用 harmonic。
- **Memory-bound vs Compute-bound 优化方向相反**：memory-bound 要减少 DRAM 访问（cache blocking），compute-bound 要提高计算并行度（vectorization）；搞反了优化没效果。
- **AI 的分母是 DRAM bytes**，不是 L1/L2 bytes（除非用 Hierarchical Roofline）。

---

## 5. 这份 slides 最可能出的题型

### 题型 1：Amdahl's Law 计算

- **识别**：给定某部分代码占总时间百分比，问优化该部分后 overall speedup 是多少；或问 speedup 上限。
- **解题步骤**：
  1. 确认 $p$（被优化部分的时间比例，是小数）
  2. 确认 $s$（该部分的加速比）
  3. 代入 $S = 1 / [(1-p) + p/s]$
  4. 求 $s\to\infty$ 时的上限：$S_{\max} = 1/(1-p)$

### 题型 2：Roofline Model 分析

- **识别**：给定 CPU spec（Peak FLOPS、DRAM BW），给定一段代码，问 memory-bound 还是compute-bound，问 attainable FLOPS，问如何优化。
- **解题步骤**：
  1. 数代码中的 FLOP 数量（每次循环迭代的浮点加减乘除）
  2. 数 DRAM 访问的 bytes（内存读写次数 × 数据类型字节数：DP=8B，SP=4B，HP=2B）
  3. 算 AI = FLOP / bytes
  4. 算 crossover = Peak FLOPS / Peak BW
  5. 比较 AI vs crossover，判断类型
  6. Attainable FLOPS = min(Peak FLOPS, BW × AI)

### 题型 3：Strong/Weak Scaling 判断

- **识别**：给定场景描述，问属于哪种 scaling。
- **解题**：问题大小固定 → Strong；问题大小随进程数增加 → Weak。

### 题型 4：平均值选择

- **识别**：给定多组运行时间或吞吐量，要求计算平均。
- **解题**：时间/speedup 比值 → geo mean；吞吐量固定 #ops → harmonic mean；执行时间 → arithmetic mean。

### 题型 5：时间类型分析

- **识别**：给定 `time` 命令输出，问各项含义；或问 real vs CPU 哪个更大。
- **解题**：real = 挂钟时间（含 I/O 等待）；CPU = user + sys；real ≥ CPU。

### 题型 6：Profiling 分析

- **识别**：给 perf annotate 输出，问为什么某几条指令占了大部分时间，或为什么采样 1 第误差大。
- **解题**：采样数少 → 统计误差大（误差 ∝ 1/N），需要更多迭代次数以获得准确时间归因。

---

## 6. 一页速记版（考前最后看）

**三步法**：Measure → Analyze → Optimize

**Amdahl's Law**：$S = 1/[(1-p)+p/s]$，上限 $1/(1-p)$

**Scaling Efficiency**：$E = t_{\text{serial}} / (t_{\text{parallel}} \times p)$

**Strong Scaling** = 问题固定，加机器提速；**Weak Scaling** = 问题随机器等比增大

**AI** = FLOP / DRAM bytes；crossover = Peak FLOPS / Peak BW
- AI < crossover → **memory-bound** → cache blocking
- AI > crossover → **compute-bound** → vectorization

**Peak FLOPS** = cores × freq × FLOP/cycle

**平均值**：时间/比值 → geo；吞吐量(固定#ops) → harmonic；时间 → arithmetic

**延迟从近到远**：Register(1)→L1(3)→L2(10)→L3(75)→DRAM(100)→IB(2000+)→PCIe(5000+)→Ethernet(20000+) cycles

**时间**：real ≥ user + sys；real = user + sys + Non-CPU

**Profiling** = 统计采样（IRQ），低开销，近似；**Tracing** = 精确插桩，高开销，全量

**FP32**：1+8+23 bits；**FP64**：1+11+52 bits；精度由尾数位决定

---

## 7. 自测题（附简短答案）

1. **已知某程序 80% 的时间在矩阵乘法上，将其优化 4 倍，overall speedup 是多少？上限是多少？**
   - 答案：$S = 1/[0.2 + 0.8/4] = 1/[0.2+0.2] = 2.5$；上限 $= 1/0.2 = 5$

2. **CPU Peak FLOPS = 100 GFLOPS，DRAM BW = 50 GB/s，crossover AI 是多少？**
   - 答案：$100 / 50 = 2$ FLOP/byte

3. **DAXPY 代码 `Z[i] = A*(X[i]+Y[i])`，X/Y/Z 均为 FP64（8 bytes），每次迭代 2 FLOP，3 次 DRAM 读写，AI = ？是 memory-bound 还是 compute-bound（crossover = 2）？**
   - 答案：AI = 2 FLOP / (3×8 bytes) ≈ 0.083 FLOP/byte；0.083 < 2 → **memory-bound**

4. **Strong Scaling 和 Weak Scaling 各自用 Scaling Efficiency 怎么看好坏？**
   - 答案：$E$ 越接近 1 越好。Strong scaling 中随 $p$ 增大 $E$ 下降，Weak scaling 中理想情况时间不变则 $E$ 保持 1。

5. **`time ./a.out` 输出 real=5s, user=3s, sys=0.5s，Non-CPU time 是多少？**
   - 答案：Non-CPU = real − (user + sys) = 5 − 3.5 = **1.5s**

6. **为什么 Profiling Sampling 1（循环 1M 次）时间只分配给 2 条指令，而 Sampling 2（循环 1B 次）分配更合理？**
   - 答案：循环次数少 → 采样点少（IRQ 次数少）→ 统计误差大，采样结果随机集中在某几条指令；循环更多 → 采样点多 → Monte Carlo 误差 ∝ 1/N 缩小，时间归因更准确。

7. **对 5 组 speedup 数据 [1.5, 2.0, 3.0, 0.5, 4.0] 取平均，应用哪种均值？结果大约是多少？**
   - 答案：Geometric mean（因为是比值/speedup）；$(1.5 \times 2.0 \times 3.0 \times 0.5 \times 4.0)^{1/5} = 18^{0.2} \approx 1.78$

8. **Cache Blocking 优化后，程序在 Roofline 图上的点如何移动？**
   - 答案：减少了 DRAM bytes 访问（cache 过滤了部分 DRAM 访问），AI 增大，点向右移；可能从 memory-bound 区域移入 compute-bound 区域，attainable FLOPS 提升。