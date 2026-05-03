# Virtualisation Basics

**Path:** Pre-Security → Computer Fundamentals
**Room:** [Virtualisation Basics](https://tryhackme.com/room/virtualisationbasics)
**Difficulty:** Easy · **Time:** ~30 min
**Completed:** 2026-05-03

## What the room covered

Why virtualisation exists, what a hypervisor does, the difference between VMs and containers, and a hands-on simulation of managing a virtual environment (restarting a broken VM, creating a new one, reading host capacity).

The five tasks: intro, virtualization overview (the "one server = one application" problem), virtualization components (hypervisors, VMs, containers), managing VMs through a fake "Virtualization Manager" web app, and a conclusion.

## Key concepts

**The pre-virtualisation problem.** One server per application meant high cost, low utilisation (5–20% CPU on most boxes), slow deployment, and painful scaling.

**Hypervisor.** Software that splits one physical machine into many virtual ones, allocates CPU/RAM/storage to each, and keeps them isolated.
- Type 1 — runs directly on hardware. Fast and efficient. Used for production servers, databases, data centers.
- Type 2 — runs on top of an existing OS (VirtualBox, VMware Workstation). Easier to install. Used for learning, testing, malware analysis, running Kali on your laptop.

**Virtual Machine.** Acts like a real computer with its own virtual CPU, RAM, disk, and network. Fully isolated — one VM crashing doesn't take down the others. Can run any OS.

**Container.** A lightweight isolated environment for a single app and its dependencies. Shares the host kernel, so it starts almost instantly and uses fewer resources than a VM — but the container OS type has to match the host (no Windows containers on Linux). Docker is the common tool.

**Mental model.** Building = physical server. Apartments = VMs. Rooms inside an apartment = containers. Building manager = hypervisor.

**Malware testing caveat.** When using a VM to detonate malware, run a different OS than the host or fully isolate the guest network so the host can't get infected.

**Hands-on (AutoGalo lab).** Restarted a Mail-SERVER VM stuck in error state, created a new Marketing-VM (4 cores / 8 GB RAM / 100 GB disk), reviewed host capacity across HV-PROD-01, HV-PROD-02 (near 100%), and HV-BACKUP-01 (disconnected).

## What clicked

The AutoGalo hands-on. Reading about hypervisors and VMs was abstract — actually opening the Virtualization Manager, finding Mail-SERVER stuck in an error state and restarting it, then creating Marketing-VM by filling in CPU/RAM/disk, made the whole "you allocate resources from a pool" idea concrete. Same with the host view — seeing HV-PROD-02 near 100% capacity and HV-BACKUP-01 disconnected showed why someone needs to actually watch this stuff.

## What to revisit

Type 1 vs Type 2 hypervisors. The basic distinction is clear (Type 1 runs on bare metal, Type 2 runs on top of an OS), but I want to drill the use-case table — which type fits production servers, data centers, malware testing, software testing, Kali, databases — so I can answer it without thinking. Also worth re-reading why Type 1 is faster (no host OS overhead) and what specific products are Type 1 (ESXi, Hyper-V, Xen) vs Type 2 (VirtualBox, VMware Workstation).
