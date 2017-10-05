# AWS-SysOps-Docs
### Monitoring & Metrics
#### Virtualization Types
HVM AMIs (Hardware virtual machine):
- Can use special hardware extensions
- Can use PV drivers for network and storage
- Usually the same or better performance than PV alone

PV AMIs (Paravirtual):
- Historically faster than HVM, but no longer the case

#### Demonstrate Ability to Monitor Availability and Performance
#### 1.Understanding AWS Instance Types, Utilization, and Performance
Instance Types – General Purpose

T2 instances:
- Intended for work loads that do not use the full CPU often or consistently
- Provide Burstable Performance
- EBS-only storage

M3 instances:
- Provide a balance of compute, memory, and network resources
- SSD Storage (Instance store)

M4 instances:
- Provide a balance of compute, memory, and network resources
- Support Enhanced Networking
- EBS-optimized

Instance Types – Compute Optimized

- Lowest price/compute performance in EC2

C3 instances
- SSD-backed instance storage
- Support for Enhanced Networking and Clustering

C4 instances
- Latest generation of Compute-optimized instances
- Highest performing processors (optimized specifically for EC2)
- Support for Enhanced Networking and Clustering
- EBS-optimized

Instance Types – Memory Optimized

- Lowest price per amount (GiB of RAM) and memory performance

R3 instances
- SSD-backed instance storage
- High memory capacity
- Support for Enhanced Networking

Instance Types – GPU

Graphics and general purpose GPU compute

G2 instances
- High frequency processors
- High-performance NVIDIA GPUs
- On-board hardware video encoder
- Low-latency frame capture and encoding, enabling interactive streaming
- Useful for GPU compute workloads, machine learning, video encoding, 3D   
application streaming, etc…

Instance Types – Storage Optimized

- Very fast SSD-backed instance storage optimized for high random I/O performance and high IOPS (I/O operations per second)
- I2 instances
  - High I/O performance (including high random performance)
  - High frequency processors
  - SSD storage
  - Supports TRIM
  - Supports Enhanced Networking
  
Instance Types – Burstable Performance
-CPU Credits are used to “burst” past the baseline performance up to 100% of a  CPU core

2.EC2 Instance & System Status Checks
System Status Checks
-Loss of network connectivity
-Loss of system power
-Software issues on the physical host
-Hardware issues on the physical host
Solution:
    Stop and start instances
    Terminate and re-launch instances
    Contact AWS
Instance Status Checks
-Failed system status checks
-Incorrect networking or startup configuration
-Exhausted memory
-Corrupted file system
-Incompatible kernel
Solution:
    Solve what is causing the issue
    Stop and start instances
    Terminate and re-launch instances with more memory, a different kernel, or different networking configuration
