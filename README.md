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

## 📊 Results Verification

The hardware takes frequency domain inputs and maps them into composite 16-bit packed spatial vectors. By converting the output Hex values (2's complement) to signed decimals, I verified the IFFT math against the expected results.
### INPUT
![image](https://github.com/shreyasingh2302vl10/IFFT_using_IP_Core/blob/de7755eff34aa1724d1672378992a7e19f0680b1/Screenshot%202026-06-10%20023900.png)
### OUTPUT
![image](https://github.com/shreyasingh2302vl10/IFFT_using_IP_Core/blob/de7755eff34aa1724d1672378992a7e19f0680b1/Screenshot%202026-06-10%20023936.png)
This project demonstrates the implementation of an **8-Point Inverse Fast Fourier Transform (IFFT)** using the Xilinx FFT IP Core in Vivado.

---

## 📋 Project Journey

I built this project to understand how FPGAs perform complex math like IFFT using Xilinx IP cores. Here are the steps I followed:

### 1. Setting up the FFT Core
I used the Xilinx FFT IP core from the Vivado Catalog. I configured it for an 8-point transform and chose the "Radix-2 Burst I/O" architecture because it is very efficient for FPGA resources. I set the data format to "Fixed Point" to keep the math precise.

### 2. Connecting the Data (AXI-Stream)
The IP core uses the AXI4-Stream protocol. I created a 16-bit wide bus where the bottom 8 bits carry the "Real" numbers and the top 8 bits carry the "Imaginary" numbers.

### 3. Creating the Testbench
I wrote a Verilog testbench to feed data into the core. I created a sequence of 8 input numbers (`100, 50, 0, 20, 0, 10, 0, 80`) and pushed them into the IP core one by one. I used the `tlast` signal to tell the core when the 8th sample was sent.

### 4. Analyzing the Simulation
After running the simulation, I noticed a "gap" in the output. This is normal because the "Burst I/O" mode takes time to calculate everything internally before sending the results out. I then decoded the hexadecimal output (like `d864`) into signed decimal numbers to verify that the IFFT math was correct.

### 5. Saving the Code
Finally, I initialized a Git repository, committed all my files, and pushed the entire project to GitHub to keep it safe and professional.

---

##  Technical Specifications

| Feature | Configuration |
| :--- | :--- |
| **Transform Length** | 8 Points |
| **Architecture** | Radix-2 Burst I/O |
| **Data Format** | Fixed Point |
| **Input Data Width** | 16-bit (8-bit Real + 8-bit Imaginary) |
| **Clock Frequency** | 50 MHz |

---

## How to Run
1. Clone this repository: `git clone https://github.com/shreyasingh2302v110/IFFT_using_IP_Core.git`
2. Open in Vivado.
3. Set `tb_top_system.v` as the top module.
4. Run Behavioral Simulation.
