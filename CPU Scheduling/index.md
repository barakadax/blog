# CPU Scheduling
- [What it is](#what-it-is)
- [What is a Process](#what-is-a-process)
- [Preemption vs Non-Preemption](#preemption-vs-non-preemption)
- [Algorithms by Class](#algorithms-by-class)
- [Scheduling in Containers & Kubernetes](#scheduling-in-containers--kubernetes)
- [Sources](#sources)

## What it is

CPU Scheduling is the mechanism by which an operating system allocates CPU execution time to competing processes.
It aims to optimize resources by maximizing CPU utilization and throughput while minimizing waiting, response, and turnaround times.

## What is a Process

A process is an active instance of a computer program in execution.
It contains the program code, its current execution state (tracked by the program counter and registers), and its allocated system resources (such as memory space, file descriptors, and security contexts).
While a program is a passive set of instructions on disk, a process is the dynamic execution entity managed by the operating system scheduler.

## Preemption vs Non-Preemption

- **Non-Preemptive**: A running task retains control of the CPU until it voluntarily terminates or blocks (e.g., waiting for I/O).
- **Preemptive**: The operating system scheduler can interrupt a running task to reallocate the CPU to another task based on priorities, time slices, or system events.

## Algorithms by Class

### Non-Preemptive
| Algorithm | Core Concept | Primary Benefit | Main Drawback / Trade-off |
| :--- | :--- | :--- | :--- |
| **FCFS** (First-Come, First-Served) | Tasks run strictly in order of arrival. | Simple to write; zero tracking overhead. | Convoy Effect: Short tasks get stuck behind massive ones. |
| **SJF** (Shortest Job First) | Selects the task with the shortest next CPU burst time. | Minimizes average waiting time. | Causes starvation; requires predicting execution length. |
| **Cooperative** | Tasks run until they voluntarily yield control or block. | Simple, predictable; avoids hardware interrupts. | One misbehaved task can freeze the entire system. |

### Preemptive
| Algorithm | Core Concept | Primary Benefit | Main Drawback / Trade-off |
| :--- | :--- | :--- | :--- |
| **SRTF** (Shortest Remaining Time First) | Preempts execution if a newly arrived task has a shorter remaining burst. | Optimizes throughput for short jobs. | High starvation risk for long-running processes. |
| **Round Robin (RR)** | Rotates tasks through a fixed time quantum (slice). | Absolute fairness; excellent interactive response. | Mismatched quantum sizes create high overhead or lag. |
| **Priority Scheduling** | Executes tasks based on an assigned priority level (can be preemptive or non-preemptive). | Crucial system operations take precedence. | Lower priority tasks can starve indefinitely (requires Aging). |
| **Fixed-Priority Preemptive (FPPS)** | Schedules tasks based on static priority rankings, preempting lower priorities immediately. | Guarantees execution of critical tasks. | Highly susceptible to priority inversion and starvation. |

### Hybrid / Advanced
| Algorithm | Core Concept | Primary Benefit | Main Drawback / Trade-off |
| :--- | :--- | :--- | :--- |
| **Multilevel Queue** | Splits tasks permanently into separate queues based on type (can be preemptive or non-preemptive). | Easily isolates background batch work from UI. | Static and inflexible; tasks cannot shift queues. |
| **MLFQ** (Multilevel Feedback Queue) | Dynamically shifts tasks between queues based on runtime habits. | Automatically learns to favor interactive tasks. | Highly complex to configure and balance. |

### Proportional Share
| Algorithm | Core Concept | Primary Benefit | Main Drawback / Trade-off |
| :--- | :--- | :--- | :--- |
| **CFS** (Completely Fair Scheduler) | Balances virtual runtime (`vruntime`) in a Red-Black tree structure. | Predictable, stable mathematical fairness. | Interactive tasks can suffer wake-up latency. |
| **EEVDF** (Earliest Eligible Virtual Deadline First) | Schedules tasks based on eligible virtual deadlines. | Reduces latency spikes for network/audio tasks. | Complex internal math mechanics. |
| **Lottery Scheduling** | Randomly draws allocated tickets to choose the next task. | Highly customizable; simple to scale dynamically. | Short-term resource allocation is highly chaotic. |
| **Stride Scheduling** | Schedules tasks based on "strides" inversely proportional to their resource share. | Deterministic, precise share allocation without lottery randomness. | Still requires tracking state/passes per task. |

### Real-Time
| Algorithm | Core Concept | Primary Benefit | Main Drawback / Trade-off |
| :--- | :--- | :--- | :--- |
| **Rate-Monotonic (RM)** | Static real-time policy; shorter periodic cycles get higher priority. | Provably optimal for fixed real-time environments. | Maximum safe CPU utilization caps out around 69%. |
| **EDF** (Earliest Deadline First) | Dynamic real-time policy; closest deadline runs next. | Maximizes CPU utilization up to 100%. | Collapses completely if the system becomes overloaded. |

### Extensible / Desktop
| Algorithm | Core Concept | Primary Benefit | Main Drawback / Trade-off |
| :--- | :--- | :--- | :--- |
| **SCX_LAVD** (Valve/Meta) | Loaded via eBPF; tracks waker-wakee execution dependency trees. | Dramatically reduces tail latency in gaming and servers. | Demands modern kernel modules (`sched_ext`). |
| **BORE** (Burst-Oriented Response Enhancer) | Scales time slices by mapping a task's inherent "burstiness." | Eliminates micro-stuttering under mixed system loads. | Departs from pure strict fairness models. |
| **BFS** (Brain Fuck Scheduler) | Uses a single queue with deadline-based lookup to prioritize low-latency interactivity. | Extremely responsive for desktop/gaming; no queue load-balancing overhead. | Does not scale well to high-core count servers. |

### Enterprise & Specialized
| Algorithm | Core Concept | Primary Benefit | Main Drawback / Trade-off |
| :--- | :--- | :--- | :--- |
| **IBM z/OS WLM** | Scales resources dynamically based on user-defined business goals. | Ties computational execution directly to business KPIs. | Requires proprietary hardware and software stacks. |
| **Windows NT Thread Scheduler** | Multi-level feedback queue using explicit foreground priority boosts. | Excellent desktop interactivity and snappy UI feel. | Can degrade background server efficiency on client OS. |
| **NVIDIA vGPU Scheduler** | Distributes GPU time slices (Equal or Fixed) across virtual machines. | Prevents resource hogging on shared accelerator hardware. | Fixed mode wastes compute cycles if a VM sits idle. |
| **O(n) Scheduler** | Scans all active tasks to calculate priorities and time slices on every decision. | Simple to implement for early multi-core systems (Linux 2.4). | Performance degrades linearly ($O(n)$) as active tasks increase. |
| **O(1) Scheduler** | Uses active/expired arrays and bitmasks to achieve constant-time scheduling. | Highly scalable ($O(1)$ complexity) for large servers (Linux 2.6). | Complex interactive heuristics; prone to audio/desktop stutter. |
| **FreeBSD ULE** | Multi-queue scheduler mapping tasks to specific CPUs with separate interactive/batch queues. | High cache affinity; excellent SMP performance and responsiveness. | High implementation complexity. |

### Hardware-Guided
| Algorithm | Core Concept | Primary Benefit | Main Drawback / Trade-off |
| :--- | :--- | :--- | :--- |
| **AMD CPPC & 3D V-Cache Optimizer** | Uses firmware data to route threads to either Cache or Frequency CCDs. | Maximizes performance for asymmetric multi-die setups. | Requires specialized hardware-driver integration. |
| **Intel Thread Director (HFI)** | Monitors hardware instruction mixes to shift threads between P/E-cores. | Precise core allocation based on real-time task needs. | Adds deep hardware-microcontroller cross-dependency. |

## Scheduling in Containers & Kubernetes

Although developers often define CPU and memory limits inside container configurations, CPU scheduling cannot be controlled from within a container or a `Dockerfile`.
Because containers are not virtual machines, they do not run their own guest kernels, they are simply isolated namespaces and control groups (cgroups) executing directly on the host operating system.
As a result, containerized threads are scheduled entirely by the host's kernel scheduler.

To achieve custom CPU scheduling (such as utilizing `scx_lavd` or `BORE`) for containerized workloads, modifications must be made at the infrastructure layer rather than the application layer.
In Kubernetes, this is typically implemented via one of two methods:

1. **Dynamic eBPF Orchestration (`sched_ext`)**: For worker nodes running modern kernels (Linux 6.12+) that support extensible scheduling, a cluster-wide DaemonSet (using tools like *Gthulhu*) can dynamically load and enforce scheduling policies at the kernel level via eBPF.
2. **Node Tainting & Labeling**: Specific worker nodes can be booted with specialized host kernels, labeled in the cluster (e.g., `scheduler=lavd`), and targeted by latency-critical workloads using a `nodeAffinity` block in the deployment configuration:

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: scheduler
            operator: In
            values:
            - lavd
```

## Sources

- [Wikipedia - Scheduling (computing)](https://en.wikipedia.org/wiki/Scheduling_(computing))
- [sched_ext: Extensible Scheduler Class (Linux Kernel)](https://docs.kernel.org/scheduler/sched-ext.html)
