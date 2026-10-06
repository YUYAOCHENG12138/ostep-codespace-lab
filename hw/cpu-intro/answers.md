## Q1
- Prediction / 预测:总时间：10 tick，CPU利用率100%
- Reasoning / 理由:PID0和PID1均为纯CPU指令，没有I/O操作。每条CPU指令消耗1 tick，总指令数5+5=10。CPU全程工作无空闲。
- Verified result / 验证结果:总时间：10 tick，CPU利用率100%
- Analysis / 分析:预测结果与模拟器运行结果一致。两个纯CPU进程交替执行，CPU全程无空闲时间。

## Q2
- Prediction / 预测:总时间：11 tick，CPU Busy 6 tick
- Reasoning / 理由:PID0有4条CPU指令，占用4 tick；PID1执行一次完整I/O需要7 tick(1+5+1)。默认调度，先跑完PID0，再运行PID1，总时间4+7=11 tick。CPU占用为PID0的4 tick加上I/O前后的2 tick，合计6 tick。
- Verified result / 验证结果:总时间：11 tick，CPU Busy 6 tick（CPU利用率54.55%）
- Analysis / 分析:预测与运行结果一致。PID1进行I/O阻塞的5个tick期间没有就绪进程，CPU空闲，因此CPU利用率不高。

## Q3
- Prediction / 预测:总时间：9 tick，CPU Busy 4 tick
- Reasoning / 理由:PID0、PID1都执行一次I/O。PID0发起I/O后切换到PID1，PID1随即也发起I/O；两个进程同时进入I/O阻塞，阻塞共5 tick；之后依次完成io_done。总时间 1+1+5+1+1 =9 tick；CPU忙碌一共4 tick。
- Verified result / 验证结果:总时间：8 tick，CPU Busy 4 tick（CPU利用率50.00%）
- Analysis / 分析:预测总时间出错，错误地把I/O阻塞时间叠加计算。I/O可以并行处理，两个进程的I/O阻塞时间发生重叠，不是先后执行，因此总时间缩短为8 tick。CPU忙碌时间和预测一致。

## Q4
- Prediction / 预测:总时间：21 tick，CPU Busy 11 tick
- Reasoning / 理由:PID0一共执行3轮I/O，每一轮I/O包含发起io(1tick)、I/O阻塞(5tick)、io_done(1tick)。第一轮I/O阻塞期间，CPU并行运行PID1的5条CPU指令，PID1在此阶段全部执行完毕。后面两轮I/O阻塞阶段没有就绪进程，CPU空闲。CPU忙碌时间为PID0每轮2tick共6tick加上PID1的5tick，合计11 tick。
- Verified result / 验证结果:总时间：21 tick，CPU Busy 11 tick（CPU利用率52.38%）
- Analysis / 分析:预测结果与仿真一致。PID0第一轮I/O阻塞期间，CPU并行执行PID1的全部CPU任务，实现CPU与I/O重叠；后两轮I/O阻塞没有就绪任务，CPU空闲。

## Q5
- Prediction / 预测:总时间：26 tick，CPU Busy 11 tick
- Reasoning / 理由:调度策略SWITCH_ON_END：只有进程结束才切换CPU，发起I/O不会发生切换。PID0占有CPU完成全部3轮I/O；每一轮I/O阻塞时CPU直接空闲，无法运行PID1。PID0全部结束后CPU才调度PID1执行5条CPU指令。CPU忙碌总tick数不变。
- Verified result / 验证结果:总时间：26 tick，CPU Busy 11 tick（CPU利用率42.31%）
- Analysis / 分析:预测与仿真结果一致。SWITCH_ON_END策略下，发起I/O不切换CPU，PID0进行I/O阻塞时CPU空闲，无法并行执行PID1；必须等PID0全部执行结束，PID1才能运行。相比Q4总时间变长，CPU利用率下降。

## Q6
- Prediction / 预测:总时间：9 tick，CPU Busy 4 tick
- Reasoning / 理由:使用IO_RUN_LATER策略，I/O完成后进程被放到就绪队列尾部，不会立刻抢占CPU。PID0、PID1先后发起I/O，同时阻塞；I/O结束后，按照就绪队列顺序依次执行io_done。CPU忙碌4 tick。
- Verified result / 验证结果:总时间：8 tick，CPU Busy 4 tick（CPU利用率50.00%）
- Analysis / 分析:CPU忙碌tick数预测正确；总时间预测偏差1tick。两个进程t1、t2先后发起IO，t3‑t7为IO阻塞周期；t7执行PID0的io_done，t8执行PID1的io_done。IO_RUN_LATER策略，IO完成进程加入就绪队列尾部，按顺序调度运行io_done。

## Q7
- Prediction / 预测:总时间：8 tick，CPU Busy 4 tick
- Reasoning / 理由:IO_RUN_IMMEDIATE策略，I/O一旦完成，对应进程立刻抢占CPU。PID0、PID1同时结束I/O，最先完成I/O的进程立即执行io_done，随后调度另一个进程。CPU忙碌4 tick。
- Verified result / 验证结果:总时间：8 tick，CPU Busy 4 tick（CPU利用率50.00%）
- Analysis / 分析:预测和仿真结果完全吻合。本场景两个进程I/O同时结束，IO_RUN_IMMEDIATE与IO_RUN_LATER得到完全相同的总耗时。只有当I/O完成时刻错开时，两种I/O调度策略的性能差异才会显现。

## Q8
- Prediction / 预测:`-S`切换策略（SWITCH_ON_IO / SWITCH_ON_END）对性能影响更大。SWITCH_ON_IO允许发起I/O时切换CPU，实现CPU与I/O并行，减少总时间、提高CPU利用率；`-I`只控制I/O完成后的调度时机，对整体性能影响较小。
- Reasoning / 理由:`-S`控制**发起I/O瞬间是否让出CPU**。SWITCH_ON_IO：进程发出I/O就切换CPU，其他进程可以利用I/O等待时间运行，实现CPU‑I/O重叠。SWITCH_ON_END：必须等进程全部结束才切换，I/O阻塞阶段CPU空闲浪费。
`-I`仅决定I/O完成后进程是立刻运行还是放入就绪队列尾部，只会在I/O完成的一瞬间产生调度差异，对整体性能影响有限。
- Verified result / 验证结果:种子1：
  SWITCH_ON_IO+IO_RUN_LATER：总时间=15，CPU Busy=8（53.33%）
  SWITCH_ON_IO+IO_RUN_IMMEDIATE：总时间=15，CPU Busy=8（53.33%）
  SWITCH_ON_END+IO_RUN_LATER：总时间=18，CPU Busy=8（44.44%）

种子2：
  SWITCH_ON_IO+IO_RUN_LATER：总时间=16，CPU Busy=10（62.50%）
  SWITCH_ON_IO+IO_RUN_IMMEDIATE：总时间=16，CPU Busy=10（62.50%）
  SWITCH_ON_END+IO_RUN_LATER：总时间=30，CPU Busy=10（33.33%）

种子3：
  SWITCH_ON_IO+IO_RUN_LATER：总时间=18，CPU Busy=9（50.00%）
  SWITCH_ON_IO+IO_RUN_IMMEDIATE：总时间=17，CPU Busy=9（52.94%）
  SWITCH_ON_END+IO_RUN_LATER：总时间=24，CPU Busy=9（37.50%）
- Analysis / 分析:对比三组随机种子实验数据：
仅修改`‑I`参数（IO_RUN_LATER ↔ IO_RUN_IMMEDIATE），总运行时间、CPU利用率几乎没有变化；
把`‑S`切换为SWITCH_ON_END后，三组实验全部出现总时间大幅变长，CPU利用率明显下降。

