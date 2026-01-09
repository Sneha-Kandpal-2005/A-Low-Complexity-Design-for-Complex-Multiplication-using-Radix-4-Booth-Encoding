This project focuses on the design and FPGA implementation of a high-performance complex multiplier, optimized for Digital Signal Processing (DSP) applications like 5G wireless systems and Fast Fourier Transforms (FFTs).

## Project Overview
Traditional complex multipliers often face bottlenecks due to high latency from multi-stage additions or excessive hardware area requirements. This implementation addresses these issues by using a parallel pipeline architecture that integrates Radix-4 Modified Booth Encoding (MBE) with a fused Carry-Save Adder Tree (CSAT).

## Technical Specifications
1. **Hardware Description Language**: Verilog HDL.
2. **Design Methodology**: Parallel pipelining to increase throughput and reduce critical path delay.
3. **Parameterization**: Supports word lengths of $N = 8, 16,$ and $32$ bits.
4. **Target Hardware**: Optimized for FPGA implementation, specifically Zynq-7000.

## Key Features
1. **Radix-4 Modified Booth Encoding (MBE)**: Reduces the number of partial products by 50%, significantly lowering the computational load.

2. **Fused Carry-Save Adder Tree (CSAT)**: Compresses partial products in a single stage, eliminating the need for slow intermediate vector-merging adders.

3. **High Efficiency**: Targeted for low-complexity and high-speed performance in real-time communication environments.

## Implementation & Verification
1. **Simulation**: The design was rigorously verified through functional simulations to ensure mathematical accuracy across all parameterized bit-widths.

2. **Scalability**: The architecture is designed to be easily scaled for different precision requirements in various DSP tasks.
