Telemetry Project – UDP Client/Server System
Project Overview

This project implements a UDP-based telemetry system where a client periodically sends telemetry data (sensor readings, timestamps, and sequence numbers) to a server.
The server logs received packets and computes performance metrics such as latency, packet loss, duplication, batching efficiency, and throughput.
Network impairments (loss, duplication, jitter) are emulated using Linux tc netem, and packet behavior is validated using PCAP captures.

Directory Contents
Core Programs

Client.py – Telemetry client that generates and sends UDP packets

Server.py – UDP server that receives packets and logs them to CSV

Analysis & Plotting

calc.py – Computes metrics from received_data.csv

graphs.py – Plots loss/duplication experiment results

graph1.py – Plots interval-related results

best_N.py – Batch size performance evaluation

Experiment Scripts

baseline.sh – Baseline experiment (no network impairment)

loss_test.sh – Packet loss experiments

loss_dup_graph.sh – Loss + duplication sweep

jitter_test.sh – Jitter experiments

interval_graph.sh – Reporting interval sweep

Data & Logs

received_data.csv – Raw server-side packet logs

results.csv – Aggregated experiment metrics

*.pcap – Wireshark packet captures

baseline.pcap

loss_5percent_test.pcap

jitter_test.pcap

netem_commands_*.log – Applied tc netem configurations

Build / Setup Instructions
Requirements

Python 3.8+

Linux / WSL (required for tc netem)

Python packages:

pip install matplotlib numpy psutil

Running the System
1. Start the Server
python3 Server.py


The server listens for UDP telemetry packets and logs them to:

received_data.csv

2. Start the Client
python3 Client.py --interval 0.1


Arguments

--interval : Telemetry reporting interval in seconds

The client runs continuously until manually stopped.

3. Run Automated Experiments

Examples:

./baseline.sh
./loss_test.sh
./jitter_test.sh
./interval_graph.sh


These scripts:

Apply network conditions using tc netem

Run client and server for a fixed duration

Generate CSV metrics and plots automatically

Batching Decision

To improve network efficiency, the client supports batch telemetry messages.

Multiple sensor readings are grouped into a single UDP packet (message type 3).

This reduces:

Header overhead per measurement

Packet rate

CPU processing cost

Batching was experimentally evaluated using best_N.py to determine how batch size affects throughput and latency.

Trade-off:

Larger batches → higher throughput, higher latency

Smaller batches → lower latency, higher overhead

Field-Packing Strategy

A compact binary encoding is used to minimize packet size.

Header Format (12 bytes)
!H I f B B

Field	Size	Description
Device ID	2 bytes	Assigned by server
Sequence Number	4 bytes	Monotonic counter
Timestamp	4 bytes	Seconds since client start
Message Type	1 byte	Single / control / batch
Flags	1 byte	Reserved
Payload

Sensor values packed as 32-bit floats

Batch messages concatenate multiple float values

Payload size is capped to avoid UDP fragmentation

Footer

bytes_sent field appended to support accurate throughput calculation

PCAP Validation

Wireshark PCAP files (*.pcap) were captured during experiments to validate:

Packet loss and duplication

Out-of-order delivery

Inter-arrival timing

Observations from PCAP analysis directly match the metrics calculated from CSV logs.

Output Files

received_data.csv – Per-packet telemetry logs

results.csv – Aggregated metrics per experiment

Graphs showing latency, loss, throughput, and efficiency trends

Summary

This project demonstrates a UDP telemetry system using batching and compact field packing. Controlled experiments show how batching, reporting interval, and network impairments affect performance, reliability, and efficiency.
