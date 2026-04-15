# Mini Container Runtime with Kernel Memory Monitor

## 1. Team Information

| Name | SRN |
|------|-----|
| Sufiya | PES1UG24CS919 |
| Debangana | PES1UG24CS921 |

---

## 2. Project Overview

This project implements a **minimal container runtime** with a **kernel-space memory monitoring module**.

It demonstrates:
- Process isolation using Linux namespaces
- Multi-container supervision
- Inter-process communication (IPC) via FIFOs
- Logging using a bounded-buffer producer-consumer model
- Kernel-level memory monitoring using a Loadable Kernel Module (LKM)
- Soft and hard memory limit enforcement
- CPU scheduling behavior under contention
- Clean teardown with no zombie processes

---

## 3. Repository Structure

```
.
├── engine.c        # User-space runtime & supervisor
├── monitor.c       # Kernel memory monitor (LKM)
├── monitor_ioctl.h # Shared ioctl interface
├── cpu_hog.c       # CPU workload generator
├── memory_hog.c    # Memory workload generator
├── io_pulse.c      # I/O workload (optional)
├── Makefile        # Build system
├── rootfs-base/    # Base filesystem
├── rootfs-alpha/   # Container filesystem (copy)
├── rootfs-beta/    # Container filesystem (copy)
├── logs/           # Container logs
└── README.md
```

---

## 4. Build, Load, and Run Instructions

###  Environment
- Ubuntu 22.04 / 24.04 VM
- Linux kernel with headers installed
- Root privileges required

---

#  Setup & Usage Guide

##  Step 1: Build

```bash
make clean
make
```

This builds:
- `engine` → user-space runtime  
- `monitor.ko` → kernel module  
- Workloads → `cpu_hog`, `memory_hog`

---

##  Step 2: Load Kernel Module

```bash
sudo insmod monitor.ko
```

Verify device:
```bash
ls -l /dev/container_monitor
```

Expected:
```
/dev/container_monitor
```

---

##  Step 3: Start Supervisor

```bash
sudo ./engine supervisor rootfs-base
```

---

##  Step 4: Prepare Container RootFS

```bash
cp -a ./rootfs-base ./rootfs-alpha
cp -a ./rootfs-base ./rootfs-beta
```

---

##  Step 5: Launch Containers

Open another terminal:

```bash
sudo ./engine start alpha rootfs-alpha /bin/sh
sudo ./engine start beta rootfs-beta /bin/sh
```

---

##  Step 6: List Containers

```bash
sudo ./engine ps
```

---

##  Step 7: View Logs

```bash
ls logs
cat logs/alpha.log
```

---

##  Step 8: Memory Monitoring Demo

Open a new terminal:

```bash
sudo dmesg -w
```

Run workload:

```bash
sudo ./engine start alpha rootfs-alpha /memory_hog
```

Expected output:
```
[container_monitor] Registered container=alpha ...
[container_monitor] SOFT LIMIT ...
[container_monitor] HARD LIMIT ...
```

---

##  Step 9: Scheduling Experiment

```bash
sudo ./engine start alpha rootfs-alpha /cpu_hog
sudo ./engine start beta rootfs-beta /cpu_hog
```

Then:

```bash
top
```

Expected:
- CPU shared between multiple `cpu_hog` processes

---

##  Step 10: Stop Containers

```bash
sudo pkill -9 engine
```

Verify cleanup:

```bash
ps aux | grep cpu_hog
```

Expected:
```
(no running cpu_hog processes)
```

---

##  Step 11: Inspect Kernel Logs

```bash
dmesg | tail
```

---

##  Step 12: Unload Module

```bash
sudo rmmod monitor
```

---

#  Running Workloads Inside Containers

Before launching a container, copy binaries:

```bash
cp cpu_hog rootfs-alpha/
cp memory_hog rootfs-alpha/
chmod +x rootfs-alpha/cpu_hog
chmod +x rootfs-alpha/memory_hog
```

Run:

```bash
sudo ./engine start alpha rootfs-alpha /cpu_hog
```

---

#  Features Implemented

## ✔ Multi-container Supervision
- Central supervisor managing multiple containers

## ✔ Metadata Tracking
- Track container state and PIDs via:
```bash
engine ps
```

## ✔ Logging System
- Producer-consumer bounded buffer  
- Per-container log files  

## ✔ CLI + IPC
- FIFO-based communication between CLI and supervisor  

## ✔ Memory Monitoring (Kernel)
- Soft limit warnings  
- Hard limit enforcement (`SIGKILL`)  

## ✔ Scheduling Demonstration
- CPU contention across containers  

## ✔ Clean Teardown
- No zombie processes  
- Proper resource cleanup  

---

#  Notes

- AppArmor warnings in `dmesg` can be ignored  
- Kernel module must be reloaded after reboot  
- Containers require root privileges (namespaces)

---

#  CI Build 

```bash
make -C boilerplate ci
```

---

#  Conclusion

This project demonstrates a full pipeline from:

- user-space container orchestration  
- to kernel-level resource enforcement  

Providing a simplified yet functional container runtime system.
