# 8-Point IFFT Implementation using Xilinx FFT IP Core

This repository contains the design, testbench, and simulation verification for an **8-Point Inverse Fast Fourier Transform (IFFT)** implemented using the Xilinx FFT/IFFT IP Core in Vivado. 

---

##  Design Architecture & IP Core Settings

The system is configured with the following parameters inside the Vivado IP Catalog:
* **Transform Length ($N$):** 8 Points
* **Architecture:** Radix-2 Burst I/O *(Optimized for low hardware resource/DSP usage)*
* **Data Format:** Fixed Point
* **Input Data Width:** 8 bits (Composite 16-bit AXI-Stream interface where `[7:0]` is Real and `[15:8]` is Imaginary)
* **Scaling:** Unscaled *(Outputs raw integers to maximize computational precision)*

---

## Simulation Protocol & Behavior

### 1. Timing Sequence & Latency Gap
The simulation runs on a **50 MHz clock** (20 ns clock period). 
* **Data Ingestion:** The testbench streams 8 data samples sequentially.
* **Processing Intermission (The Latency Gap):** Since the core is configured in *Radix-2 Burst I/O*, it locks internal registers to compute butterfly stages. During this recursive calculation window, the output bus drops to a silent phase (`16'h0000`) and `m_axis_data_tvalid` remains low.
* **Data Egress:** Once processing concludes, `m_axis_data_tvalid` goes high, pulsing out the transformed time-domain results.

### 2. Numerical Mapping Verification (Hex to Signed Decimal)
The hardware takes the frequency domain inputs and maps them into composite 16-bit packed spatial vectors. For example, an input sequence of `[100, 50, 0, 20, 0, 10, 0, 80]` is processed into structural 2's complement hexadecimal values on the output bus (e.g., `16'hD864`), tracking precise proportional components for both Real and Imaginary domains.

---
