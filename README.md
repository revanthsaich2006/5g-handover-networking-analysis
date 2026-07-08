# Handover Analysis in 5G Networks 📡🤖

[![OMNeT++](https://img.shields.io/badge/OMNeT%2B%2B-6.2.0-blue.svg)](https://omnetpp.org/)
[![Simu5G](https://img.shields.io/badge/Simu5G-1.4.0-orange.svg)](https://simu5g.org/)
[![INET](https://img.shields.io/badge/INET-4.5.4-green.svg)](https://inet.omnetpp.org/)
[![Python](https://img.shields.io/badge/Python-3.x-yellow.svg)](https://www.python.org/)

A comprehensive system-level simulation and analysis of transport layer protocols (**UDP**, **TCP**, and **QUIC**) during handover events in a congested 5G multi-cell environment. 

This project was developed as a Term Project for **Computer Networks (SWE3022_41)** under the guidance of **Prof. Jaehoon (Paul) Jeong** at Sungkyunkwan University (SKKU).

---

## 📖 Project Overview

Evaluating network performance during mobility is a major challenge in 5G New Radio (NR) standalone networks. High-frequency mmWave links are susceptible to blockage, requiring frequent **X2/Xn-based handovers** between base stations (gNodeBs). 

This project implements a custom discrete-event simulation model in **OMNeT++** with **Simu5G** to benchmark how transport protocols react to mobility-induced path switches and cellular congestion.

### 🎯 Key Objectives:
1. **Custom Traffic Generation**: Developed custom C++ application modules to generate constant-bit-rate traffic and measure accurate One-Way Delay (OWD), jitter, and throughput.
2. **Network Saturation Analysis**: Conducted a parameter sweep across user density (10 to 100 User Equipments) to locate the cell capacity "Knee Point".
3. **Protocol Efficiency Sweep**: Swept payload sizes (100 to 1000 Bytes) to analyze header overhead impacts.
4. **Data Pipeline Optimization**: Engineered a Python post-processing script to aggregate and analyze over 16,500 raw CSV log files in under 30 seconds.

---

## 🛠️ Simulation Architecture & C++ Design

Standard INET simulation modules wrap packets in multiple encapsulation layers, making precise timestamping difficult. We resolved this by creating a custom application-layer packet structure:

### Custom Packet Definition (`FilePacket.msg`)
```cpp
class FilePacket extends inet::FieldsChunk {
    unsigned int nframes;           // Total frames in transmission
    unsigned int IDframe;           // Sequence Number for loss detection
    simtime_t payloadTimestamp;     // Creation Timestamp (Crucial for OWD)
    unsigned int payloadSize;       // Payload padding size
}
```

### Protocol Implementation Highlights:
*   **UDP Sender (`FileSender.cc`)**: Uses a "fire-and-forget" mechanism for lowest possible latency.
*   **TCP Sender (`FileSenderTCP.cc`)**: Uses an event-driven handshake verification (`socketEstablished()`) before initiating transmission, ensuring simulation stability.
*   **QUIC Sender (`FileSenderQUIC.cc`)**: Simulates user-space congestion control and eliminates Head-of-Line (HOL) blocking.
*   **Receiver (`FileReceiver.cc`)**: Dynamically writes results per UE/run using dynamic filenames (`results/FileReceiver_Run{RunNo}_UE{UE_ID}.csv`) to enable unattended batch simulations.

---

## 📊 Key Findings & Results

A parameter sweep consisting of **300 unique simulation runs** revealed:
*   **The Congestion Flatline (<60 UEs)**: Average packet delay remains completely flat at low loads because cell resources (Physical Resource Blocks) are abundant, meaning adding users has zero marginal cost on performance.
*   **The Knee Point (~60 UEs)**: The cell saturates at ~60 active UEs.
    *   **UDP** maintains low latency but packet loss spikes from **0% to >5%** because packets are dropped during handover switching intervals.
    *   **TCP and QUIC** buffer packets during handover to guarantee delivery, preserving **0% packet loss** at the cost of latency spikes (**20ms+**).
*   **Throughput Plateau (>600 Bytes)**: When payload size exceeds 600 Bytes, header overhead (IP/UDP/RLC/MAC/PHY) becomes negligible (<10%), causing throughput efficiency to flatten out near the theoretical channel limit.

---

## 📂 Repository Contents

*   `filetransfer_multicell.zip`: Compressed archive containing the multi-cell file transfer module source code.
*   `handover_sim.zip`: Compressed archive containing the handover simulation environment.
*   `Term Project Report CN 2025319046.pdf`: Detailed scientific report with methodology, formulas, and results.
*   `.gitattributes`: Configured for Git LFS to track large assets.

---

## 🚀 Execution Guide

### Prerequisites
*   **OS**: Windows 11 / Linux
*   **Discrete Event Simulator**: [OMNeT++ v6.2.0](https://omnetpp.org/)
*   **5G Model Library**: [Simu5G v1.4.0](https://simu5g.org/)
*   **Protocol Library**: [INET Framework v4.5.4](https://inet.omnetpp.org/)

### Running the Simulation
1. Import `filetransfer_multicell` and `handover_sim` folders (extracted from the zip archives) into your OMNeT++ workspace.
2. Build the project. Run `make clean && make` if compilation issues occur.
3. In the project explorer, navigate to `simulations/nr/handover_sim`.
4. Right-click `omnetpp.ini` -> **Run As -> OMNeT++ Simulation**. Choose one of the configurations:
    *   `Config Handover-UDP` (Baseline Testing)
    *   `Config Handover-TCP` (Reliability Testing)
    *   `Config Handover-QUIC` (Modern protocol evaluation)
5. For batch CLI execution, run:
   ```bash
   opp_run -r 0-99 -u Cmdenv -c Config Handover-UDP
   ```

### Post-Processing
Run the Python script in the results folder to aggregate the log files and generate charts:
```bash
python analyze_results.py
```
