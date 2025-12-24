Here is a **clean, ready-to-submit README.md** you can use. It includes **build instructions, usage examples, batching decision, and field-packing strategy**, written in a clear academic/technical style.

---

# Telemetry Client–Server System

## Overview

This project implements a **UDP-based telemetry system** consisting of a Python client and server.
The client periodically sends telemetry packets containing sensor data, timestamps, and sequence numbers, while the server receives, processes, and logs the packets to CSV files for performance analysis (latency, loss, duplication, throughput, and batching efficiency).

---

## Build / Setup Instructions

### Requirements

* Python **3.8+**
* Linux / WSL (for `tc netem`)
* Required Python packages:

  ```bash
  pip install matplotlib numpy psutil
  ```

### Files

* `Client.py` – Telemetry data generator and sender
* `Server.py` – UDP server and packet logger
* `calc.py` – Metric calculation from CSV logs
* `graph*.py` – Plotting scripts
* `*.sh` – Experiment automation scripts

---

## Running the System

### 1. Start the Server

```bash
python3 Server.py
```

The server listens for UDP packets and logs received data to:

```
received_data.csv
```

---

### 2. Start the Client

```bash
python3 Client.py --interval 0.1
```

**Arguments**

* `--interval` : Reporting interval in seconds (e.g., `0.05`, `0.1`, `0.5`, `1.0`)

The client sends telemetry packets continuously until stopped.

---

### 3. Run Automated Experiments

Examples:

```bash
./interval_sweep.sh
./loss_sweep.sh
```

These scripts:

* Apply network conditions (loss, duplication)
* Run client/server for a fixed duration
* Collect metrics and generate plots

---

## Usage Examples

### Interval Sweep

```bash
python3 Client.py --interval 0.05
python3 Client.py --interval 0.5
```

Used to analyze how reporting frequency affects throughput and latency.

### Loss & Duplication Sweep

Uses Linux `tc netem`:

```bash
tc qdisc add dev eth0 root netem loss 2% duplicate 5%
```

---

## Batching Decision

To reduce protocol overhead and improve throughput, the client supports **batch messages** (message type 3).

* Instead of sending one sensor value per packet, multiple values are grouped into a single UDP packet.
* This reduces:

  * Header overhead per measurement
  * CPU processing cost per report
  * Packet rate on the network

Batch sizes are configurable and were experimentally evaluated to determine an optimal trade-off between throughput and latency.

**Rationale:**

> Larger batches increase throughput but may increase latency; smaller batches reduce latency but increase overhead.

---

## Field-Packing Strategy

A **compact binary packet format** is used to minimize packet size and parsing cost.

### Header Format

```text
!H I f B B
```

| Field           | Size    | Description                |
| --------------- | ------- | -------------------------- |
| Device ID       | 2 bytes | Assigned by server         |
| Sequence Number | 4 bytes | Monotonic packet counter   |
| Timestamp       | 4 bytes | Seconds since client start |
| Message Type    | 1 byte  | Single, control, or batch  |
| Flags           | 1 byte  | Reserved                   |

Total header size: **12 bytes**

### Payload

* Sensor values packed as **32-bit floats**
* Batch messages concatenate multiple float values
* Packet size is capped to prevent UDP fragmentation

### Footer

* `bytes_sent` field appended to payload to support accurate throughput calculation

---

## PCAP and Metrics Validation

Wireshark PCAP captures were used to validate:

* Packet ordering
* Loss and duplication
* Inter-arrival times

These observations match the metrics computed from server CSV logs.

---

## Output Files

* `received_data.csv` – Raw packet logs
* `results.csv` – Aggregated performance metrics
* Graphs: throughput, latency, efficiency vs parameters

---

## Summary

This system demonstrates an efficient telemetry transport using UDP, batching, and compact binary encoding. Experimental evaluation shows how batching and reporting intervals impact network efficiency, latency, and reliability.

---

If you want, I can:

* **Shorten this to a 1-page README**
* **Rewrite it in more formal academic language**
* **Adapt it exactly to your course rubric**

Just tell me.
