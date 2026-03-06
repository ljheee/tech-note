

CPU亲和性
taskset -c 2 ./myapp
affinity 设置对 SCHED_FIFO/RR 是“硬”的；但中断、软中断、ksoftirqd 仍可能偷到隔离核。


nice 值通过预定义数组 sched_prio_to_weight 映射为权重（非线性能）。
nice=-5 → 权重 1995（约 1.95 倍 CPU 时间）。
nice=0 → 权重 1024（基准值）。
nice=+5 → 权重 335（约 0.33 倍 CPU 时间）。

权重按 2^((nice)/4) 近似计算，确保平滑过渡。

vruntime += 实际运行时间 × (NICE_0_LOAD / 权重)
NICE_0_LOAD 是 nice=0 的权重（1024）。


效果：高 nice（低权重）任务的 vruntime 增长更快，被调度机会更少。


时间片分配：
在 sched_latency 周期内，任务按权重分配时间片。
时间片 = (sched_latency × 任务权重) / 所有任务权重总和
最小时间片：受 sched_min_granularity 限制（默认 0.75ms），防止时间片过短。

