# Processing units
- [What is a processing unit?](#what-is-a-processing-unit)
- [Types of processing units](#types-of-processing-units)
  - [ALU](#alu-arithmetic-logic-unit)
  - [CPU](#cpu-central-processing-unit)
  - [GPU](#gpu-graphics-processing-unit)
  - [TPU](#tpu-tensor-processing-unit)
  - [MXU](#mxu-matrix-multiply-unit)
  - [APU](#apu-accelerated-processing-unit)
  - [NPU](#npu-neural-processing-unit)
  - [DSP](#dsp-digital-signal-processor)
  - [VPU](#vpu-vision-processing-unit)
  - [ISP](#isp-image-signal-processor)
  - [HSM](#hsm-hardware-security-module)
  - [DPU](#dpu-data-processing-unit)
  - [QPU](#qpu-quantum-processing-unit)
  - [FPGA](#fpga-field-programmable-gate-array)
- [System on a Chip (SoC)](#system-on-a-chip-soc)
- [Architectures in processing units (ISA)](#architectures-in-processing-units-isa)
- [Sources](#sources)

## What is a processing unit?

A processing unit is a hardware component that performs operations on data.
It is the core of any computing system, responsible for executing instructions and processing information to produce a desired output.
Modern computing relies on a variety of specialized processing units, each optimized for specific types of workloads, from general-purpose calculations to complex artificial intelligence and graphics rendering.

## Types of processing units

### ALU (Arithmetic Logic Unit)
A fundamental digital circuit within a processor (like a CPU) that performs arithmetic operations (addition, subtraction, etc.) and bitwise logic operations.
- **Composition**: Standalone digital circuit, but primarily serves as a core building block within larger processing units.
- **Components**: Logic gates configured for arithmetic and bitwise calculations.

### CPU (Central Processing Unit)
The "brain" of the computer. It is a general-purpose processor designed to handle a wide range of tasks and instructions sequentially.
- **Composition**: Multi-part integrated system.
- **Components**: Includes arithmetic logic units (ALU), a control unit (CU), registers, and cache memory.

### GPU (Graphics Processing Unit)
Originally designed for graphics rendering, GPUs are optimized for parallel processing.
They are now widely used for general-purpose computing, especially in generative AI, machine learning, and scientific simulations.
- **Composition**: Multi-part system. Can be **standalone** (discrete graphics card) or **integrated** (iGPU) within a CPU/SoC.
- **Components**: Thousands of smaller cores (e.g., CUDA or Stream cores), video memory (VRAM), and a memory controller.

### TPU (Tensor Processing Unit)
An application-specific integrated circuit (ASIC) developed by Google to accelerate machine learning.
It is specifically designed to accelerate neural network machine learning tasks, optimized for high-volume, low-precision tensor operations.
- **Composition**: Multi-part specialized system.
- **Components**: Matrix Multiply Unit (MXU), High Bandwidth Memory (HBM), and Scalar/Vector units.

### MXU (Matrix Multiply Unit)
A specialized hardware accelerator designed to perform matrix multiplication operations.
- **Composition**: Multi-part specialized system.
- **Components**: Standalone digital circuit.

### APU (Accelerated Processing Unit)
A single integrated circuit that combines a CPU and a GPU.
- **Composition**: Multi-part integrated package.
- **Components**: CPU cores, GPU cores, and a shared memory controller on a single silicon die.

### NPU (Neural Processing Unit)
A specialized hardware accelerator designed to accelerate AI and machine learning tasks.
- **Composition**: Often integrated as a multi-part block within a host processor or SoC.
- **Components**: Neural compute engines, matrix multiplication blocks, and direct memory access (DMA) engines.

### DSP (Digital Signal Processor)
A specialized microprocessor designed to perform rapid mathematical operations on digital signals.
- **Composition**: Can be a **standalone** chip or integrated as a functional block within an SoC.
- **Components**: Compute engine (ALU/Multipliers), program/data memory, and specialized I/O.

### VPU (Vision Processing Unit)
A type of AI accelerator designed to accelerate machine vision tasks.
- **Composition**: **Standalone** device or integrated accelerator.
- **Components**: Parallel processing elements (PEs), local buffers, and specialized vision-specific ALU arrays.

### ISP (Image Signal Processor)
A specialized component that converts raw sensor data into high-quality images.
- **Composition**: Can be **standalone** (for high-end cameras) or integrated into a sensor/SoC.
- **Components**: A/D converter, image-filtering DSP, and temporary memory units.

### HSM (Hardware Security Module)
A physical computing device that safeguards digital keys and performs cryptographic functions.
- **Composition**: **Standalone** (plug-in card/external) or embedded within larger systems.
- **Components**: Secure processor, hardware-based random number generator (TRNG), and tamper-resistant memory.

### DPU (Data Processing Unit)
A specialized processor designed to offload data-centric tasks (networking, storage, security).
- **Composition**: Multi-part System-on-a-Chip (SoC).
- **Components**: Multi-core CPU, high-performance network interface, and programmable acceleration engines.

### QPU (Quantum Processing Unit)
The specialized "brain" of a quantum computer.
- **Composition**: Complex multi-part apparatus.
- **Components**: Quantum chip (qubits), control electronics, and specialized cryogenic cooling infrastructure.

### FPGA (Field-Programmable Gate Array)
A semiconductor device that can be reprogrammed after manufacturing.
- **Composition**: **Standalone** reconfigurable chip.
- **Components**: Configurable logic blocks (CLBs), programmable interconnects, and I/O blocks.

## System on a Chip (SoC)

A SoC is an integrated circuit that consolidates most components of a computer or other electronic system onto a single die.
Instead of having separate chips for the CPU, GPU, and memory, an SoC integrates them into one package to improve efficiency, reduce power consumption, and save space.
- **Key Advantage**: Faster communication between components and significantly lower power usage.
- **Common Examples**: Apple M-series chips, Qualcomm Snapdragon, and Raspberry Pi processors.
- **Components Often Integrated**: CPU, GPU, NPU, ISP, DSP, Modem, and RAM.

## Architectures in processing units (ISA)

Mostly common when reading about CPUs you will read the options are x86, x64, ARM, etc. These are **Instruction Set Architectures (ISA)**.
The differences are in the set of instructions that the CPU can execute and how they are decoded.

### RISC vs. CISC
Two fundamental design philosophies:
- **RISC (Reduced Instruction Set Computer)**: Uses a small, highly optimized set of instructions. Each instruction is simple and usually executes in one clock cycle.
    - *Examples*: **ARM** (phones, Macs), **RISC-V** (open-source), **MIPS**.
- **CISC (Complex Instruction Set Computer)**: Uses a larger set of complex instructions. A single instruction can perform multiple operations (like loading from memory and adding).
    - *Examples*: **x86** (Intel and AMD processors for PCs).

## Sources

- [Central Processing Unit](https://en.wikipedia.org/wiki/Central_processing_unit)
- [Graphics Processing Unit](https://en.wikipedia.org/wiki/Graphics_processing_unit)
- [Tensor Processing Unit](https://cloud.google.com/tpu)
- [Neural Processing Unit](https://www.ibm.com/topics/neural-processing-unit)
- [Quantum Processing Unit](https://en.wikipedia.org/wiki/Quantum_computing)
- [Instruction Set Architecture](https://en.wikipedia.org/wiki/Instruction_set_architecture)
- [System on a Chip](https://en.wikipedia.org/wiki/System_on_a_chip)
