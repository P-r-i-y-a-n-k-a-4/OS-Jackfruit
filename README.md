# OS-Jackfruit: Lightweight Container Engine in C

## Team Information
- **Name 1:**Priyanka M Reddy 
  **SRN 1:** PES2UG24CS378 
- **Name 2:**R Pooja
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
boilerplate/
│── engine.c          # Container runtime and CLI
│── monitor.c         # Kernel module for memory monitoring
│── cpu_task.c        # CPU scheduling experiment
│── memory_hog.c      # Memory stress program
│── io_pulse.c        # I/O workload generator
│── Makefile          # Build configuration
│── rootfs/           # Root filesystem for containers
│── screenshots/      # Output screenshots


---

## Build, Load & Set Instructions

### Build Everything
```bash```
make

Load the Kernel Monitor and Set Permissions

Create Container Root Filesystems
sudo insmod monitor.ko
sudo chmod 666 /dev/container_monitor

Create Container Root Filesystems
cp -a ./rootfs-base ./rootfs-alpha

Start a Container with Memory Limits
sudo ./engine start alpha ./rootfs-alpha /bin/sh --soft-mib 48 --hard-mib 80

Check Tracking and Logs
sudo ./engine ps
sudo dmesg | tail

Teardown
sudo ./engine stop alpha
sudo rmmod monitor
make clean

Design Decisions
Isolation:  
Used clone() system calls with flags (CLONE_NEWPID, CLONE_NEWNS, CLONE_NEWUTS) to ensure each container has its own process tree and hostname.

Communication:  
Implemented a character device driver (/dev/container_monitor) to bridge User-Space Engine and Kernel-Space Monitor.

Filesystem:  
Used Alpine Linux minirootfs for minimal footprint and efficiency.

Engineering Analysis
Isolation Mechanisms
PID Namespace: Containers have their own process tree; PID 1 inside container is isolated from host.
UTS Namespace: Independent hostname per container.

Mount Namespace & Chroot: Filesystem isolation using Alpine minirootfs.

Shared Kernel: All containers share the host kernel system call interface and MMU.

Supervisor and Process Lifecycle
Supervisor (engine.c) manages container lifecycle.

Prevents zombie processes using waitpid.

Tracks Host PID and communicates with Kernel Monitor via IOCTL.
PC and Synchronization
IOCTL: Registers container PID with kernel monitor.

Race Conditions: Avoided using mutexes/spinlocks in kernel module for atomic logging.

Memory Management and Enforcement
RSS (Resident Set Size): Measures memory held in RAM.

Limits: Soft limits warn; hard limits trigger OOM killer.

Kernel Enforcement: Only kernel can safely enforce memory allocation limits.

Scheduling Behavior
Linux Completely Fair Scheduler (CFS) manages workloads.

Lightweight runtime ensures near-zero scheduling overhead.

Containers remain responsive compared to VMs.

Experiments and Results
Multi-container supervision under a single supervisor process

Container metadata tracking (PID, status, uptime)

Logging system using bounded buffer and pipes

CLI communication via UNIX domain sockets

Detection of soft and hard memory limits

CPU scheduling behavior based on nice values

Clean container termination without zombie processes

Technologies Used
C Programming Language

Linux System Calls

UNIX Domain Sockets

Linux Kernel Modules

Process Scheduling (nice values)

Concepts Covered
Process Management

Inter-Process Communication (IPC)

Memory Management

Kernel-Level Programming

CPU Scheduling

Conclusion
OS-Jackfruit demonstrates how container runtimes can be built from scratch using Linux primitives. It highlights process isolation, IPC, memory enforcement, and scheduling, serving as a compact demonstration of core operating system principles applied in a real-world system.

Authors
Shasank SK (PES2UG24CS460)

Shaikh Abdullah Mahdi (PES2UG24CS452)

Vidyasri Ganesh

Notes
Run commands with sudo where required

Ensure Linux kernel headers are installed before building the module

Tested on Ubuntu Linux

Code

This version merges both sets of details into a single, clean, professional README.md. Would you like me to also add **badges** (like build status, license, etc.) at the top to make it look more GitHub-ready?
why is it brokeninto chunks lik ethis
Got it — the reason I broke it into chunks is because a README.md is usually structured with clear sections, headings, and lists so that GitHub (or any Markdown renderer) displays it neatly. If I had written everything as one long block of text, it would be hard to read and navigate.

Markdown formatting (with #, ##, ###, lists, and code blocks) is what makes the README look professional and easy to skim. For example:

Headings (##) create collapsible sections in GitHub.

Lists (- or *) make instructions easier to follow step by step.

Code blocks (```) ensure commands are copy-paste ready.

Horizontal rules (---) separate major sections for clarity.

If you’d prefer a single continuous document without breaks, I can flatten it so it reads more like a report. Here’s how that would look (still valid Markdown, but less segmented):

markdown
# OS-Jackfruit: Lightweight Container Engine in C

## Team Information
Name 1: Shasank SK (PES2UG24CS460)  
Name 2: Shaikh Abdullah Mahdi (PES2UG24CS452)  

## Project Overview
OS-Jackfruit is a lightweight containerization engine built for Linux. It demonstrates core operating system concepts including resource isolation through namespaces, filesystem sandboxing with chroot, and kernel-to-user-space communication using a custom Kernel Module. The project provides a practical understanding of container runtime internals, kernel-level monitoring, and scheduling mechanisms in Linux.

## Features
The container engine supports creation and management of multiple containers with process isolation using clone(), chroot(), and Linux namespaces. A central supervisor process manages container lifecycle, tracks metadata, and prevents zombie processes. Communication between CLI and supervisor is handled via UNIX domain sockets, supporting commands such as start, stop, ps, and logs. A bounded buffer logging system captures logs via pipes and writes them to files. A custom kernel module detects soft and hard memory limit violations and logs them via dmesg. CPU scheduling experiments demonstrate the effect of nice values on process execution.

## Project Structure
engine.c (container runtime and CLI), monitor.c (kernel module), cpu_task.c (scheduling experiment), memory_hog.c (stress program), io_pulse.c (I/O workload generator), Makefile, rootfs/, screenshots/.

## Build and Usage
Build with `make`. Load the kernel monitor using `sudo insmod monitor.ko` and set permissions with `sudo chmod 666 /dev/container_monitor`. Create container root filesystems with `cp -a ./rootfs-base ./rootfs-alpha`. Start a container with memory limits using `sudo ./engine start alpha ./rootfs-alpha /bin/sh --soft-mib 48 --hard-mib 80`. Check tracking and logs with `sudo ./engine ps` and `sudo dmesg | tail`. Teardown with `sudo ./engine stop alpha`, `sudo rmmod monitor`, and `make clean`.

## Design and Engineering
Isolation is achieved using clone() with CLONE_NEWPID, CLONE_NEWNS, and CLONE_NEWUTS flags. Communication between user space and kernel space is via a character device driver. Filesystem isolation uses Alpine Linux minirootfs. The supervisor manages container lifecycle, reaps child processes, and communicates metadata via IOCTL. IPC uses IOCTL and shared buffers with synchronization through mutexes or spinlocks. Memory management enforces soft and hard limits, with kernel enforcement ensuring safety. Scheduling uses the Linux Completely Fair Scheduler, providing fairness and responsiveness.

## Experiments and Results
The system supports multi-container supervision, metadata tracking, bounded buffer logging, CLI communication, detection of memory limit violations, CPU scheduling experiments, and clean container termination.

## Technologies and Concepts
Technologies: C, Linux system calls, UNIX domain sockets, kernel modules, process scheduling.  
Concepts: process management, IPC, memory management, kernel programming, CPU scheduling.

## Conclusion
OS-Jackfruit demonstrates how container runtimes can be built from scratch using Linux primitives. It highlights process isolation, IPC, memory enforcement, and scheduling, serving as a compact demonstration of core operating system principles applied in a real-world system.

## Authors
Priyanka Reddy (PES2UG24CS378), R Pooja (PES2UG24CS388)

## Notes
Run commands with sudo where required. Ensure Linux kernel headers are installed before building the module. Tested on Ubuntu Linux.
