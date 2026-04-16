# OS-Jackfruit: Lightweight Container Engine in C

## Team Information
- **Name 1:** Priyanka M Reddy 
  **SRN 1:** PES2UG24CS378 
- **Name 2:** R Pooja
  **SRN 2:** PES2UG24CS388

---

## Project Overview
OS-Jackfruit is a lightweight containerization engine built for Linux. It demonstrates core operating system concepts including:
- Resource isolation through **namespaces**
- Filesystem sandboxing with **chroot**
- Kernel-to-user-space communication using a **custom Kernel Module**

This project provides a practical understanding of container runtime internals, kernel-level monitoring, and scheduling mechanisms in Linux.

---

## Features

### Container Engine
- Creation and management of multiple containers
- Process isolation using `clone()`, `chroot()`, and Linux namespaces
- Execution of custom commands within containers

### Supervisor System
- Central supervisor process (`engine.c`) manages container lifecycle
- Tracks container metadata (PID, status, uptime)
- Prevents zombie processes via `waitpid`

### CLI Communication
- Communication between CLI and supervisor using UNIX domain sockets
- Supported commands:
  - `start`
  - `stop`
  - `ps`
  - `logs`

### Logging System
- Bounded buffer implementation for logging
- Logs captured via pipes
- Dedicated logging thread writes logs to files

### Memory Monitoring (Kernel Module)
- Custom kernel module (`monitor.c`)
- Detects:
  - Soft memory limit violations
  - Hard memory limit violations
- Logs generated via kernel logging (`dmesg`)

### CPU Scheduling Experiment
- Demonstrates the effect of **nice values** on process scheduling
- Shows impact of priority on execution time

---

## Project Structure

```

boilerplate/
│── engine.c          # Container runtime and CLI
│── monitor.c         # Kernel module for memory monitoring
│── cpu_task.c        # CPU scheduling experiment
│── memory_hog.c      # Memory stress program
│── io_pulse.c        # I/O workload generator
│── Makefile          # Build configuration
│── rootfs/           # Root filesystem for containers
│── screenshots/      # Output screenshots
```


---

## Build, Load & Set Instructions

### Build Everything
```bash
make
```

### Load the Kernel Monitor and Set Permissions
```bash
sudo insmod monitor.ko
sudo chmod 666 /dev/container_monitor
```

### Create Container Root Filesystems
```bash
cp -a ./rootfs-base ./rootfs-alpha
```

### Start a Container with Memory Limits
```bash
sudo ./engine start alpha ./rootfs-alpha /bin/sh --soft-mib 48 --hard-mib 80
```

### Check Tracking and Logs
```bash
sudo ./engine ps
sudo dmesg | tail
```

### Teardown
```bash
sudo ./engine stop alpha
sudo rmmod monitor
make clean
```

### Design Decisions
- Isolation
Used clone() system calls with flags (CLONE_NEWPID, CLONE_NEWNS, CLONE_NEWUTS) to ensure each container has its own process tree and hostname.
- Communication
Implemented a character device driver (/dev/container_monitor) to bridge User-Space Engine and Kernel-Space Monitor.
- Filesystem
Used Alpine Linux minirootfs for minimal footprint and efficiency.

## Engineering Analysis

### Isolation Mechanisms
- PID Namespace: Containers have their own process tree; PID 1 inside container is isolated from host.
- UTS Namespace: Independent hostname per container.
- Mount Namespace & Chroot: Filesystem isolation using Alpine minirootfs.
- Shared Kernel: All containers share the host kernel system call interface and MMU.

### Supervisor and Process Lifecycle
- Supervisor (engine.c) manages container lifecycle.
- Prevents zombie processes using waitpid.
- Tracks Host PID and communicates with Kernel Monitor via IOCTL.

### IPC and Synchronization
- IOCTL: Registers container PID with kernel monitor.
- Race Conditions: Avoided using mutexes/spinlocks in kernel module for atomic logging.

### Memory Management and Enforcement
- RSS (Resident Set Size): Measures memory held in RAM.
- Limits: Soft limits warn; hard limits trigger OOM killer.
- Kernel Enforcement: Only kernel can safely enforce memory allocation limits.

### Scheduling Behavior
- Linux Completely Fair Scheduler (CFS) manages workloads.
- Lightweight runtime ensures near-zero scheduling overhead.
- Containers remain responsive compared to VMs.

### Experiments and Results
- Multi-container supervision under a single supervisor process
- Container metadata tracking (PID, status, uptime)
- Logging system using bounded buffer and pipes
- CLI communication via UNIX domain sockets
- Detection of soft and hard memory limits
- CPU scheduling behavior based on nice values
- Clean container termination without zombie processes

### Technologies Used
- C Programming Language
- Linux System Calls
- UNIX Domain Sockets
- Linux Kernel Modules
- Process Scheduling (nice values)

### Concepts Covered
- Process Management
- Inter-Process Communication (IPC)
- Memory Management
- Kernel-Level Programming
- CPU Scheduling

### Conclusion
OS-Jackfruit demonstrates how container runtimes can be built from scratch using Linux primitives. It highlights process isolation, IPC, memory enforcement, and scheduling, serving as a compact demonstration of core operating system principles applied in a real-world system.

### Authors
- Priyanka M Reddy (PES2UG24CS378)
- R Pooja (PES2UG24CS388)

### Notes
- Run commands with sudo where required
- Ensure Linux kernel headers are installed before building the module
- Tested on Ubuntu Linux

