# Azure Notes

Personal revision notes covering Azure compute, high availability, scaling, networking, and FinOps / cost optimization.

## Topics Covered

### 1. Hierarchy & Organization
- **Management Groups:** Top-level hierarchy used for governance, RBAC, and Azure Policy compliance—not for application grouping or lifecycle management.
- **Resource Groups:** Logical containers for grouping resources that share a lifecycle, permissions, or billing/management scope.

### 2. Virtual Machines vs. RDP vs. AVD
- **Virtual Machine (VM):** Dedicated IaaS compute instance (vCPU, RAM, OS, disks) with full admin/root access and hardware isolation.
- **RDP (Remote Desktop Protocol):** Network protocol (default TCP `3389`) for remote GUI sessions. Shared/commercial RDPs lack admin rights, dedicated isolation, and production SLAs; an external admin or noisy neighbor can disrupt sessions.
- **Azure Virtual Desktop (AVD):** Microsoft's managed VDI/DaaS platform using reverse-connect over port `443` (no open inbound `3389`), Windows 11/10 Multi-Session support, FSLogix user profiles, and Scaling Plans for automated power management.
- **When to Choose What:** Use cheap/shared RDPs only for disposable, non-sensitive GUI/bandwidth tasks; use dedicated VMs for production apps, custom ports, Linux, admin control, or heavy/GPU workloads (like model training); use AVD for pooled enterprise/QA desktops.

### 3. Availability Sets & Virtual Machine Scale Sets (VMSS)
- **Availability Set:** Groups 2+ VMs (up to 200) in a single datacenter across **Fault Domains** (rack/hardware isolation, up to 3) and **Update Domains** (rolling maintenance reboots, up to 20 with a 30-minute recovery window) for a **99.95% SLA** ($0 extra charge).
  - *Gotchas:* Mixed VM families can fail allocation on the same hardware cluster; deallocating VMs before adding new ones can break FD spread; 30-minute UD wait does not check app readiness; unmanaged Availability Sets/legacy patterns are being phased out in favor of Availability Zones and VMSS Flexible.
- **Virtual Machine Scale Sets (VMSS):** Manages and scales groups of load-balanced VMs (up to 1,000 VMs, $0 extra charge) via autoscale rules, scale-in/upgrade policies, health probes, and Availability Zones/Fault Domains.
  - *Orchestration Modes (locked at creation):* **Flexible** (supports mixed VM sizes, Spot + on-demand together, standalone VM capabilities, stateful/quorum workloads) vs. **Uniform** (identical VM clones for stateless/high-scale fleets).

### 4. Azure Pricing & Cost Management (FinOps)
- **Account & Service Models:** Pay-As-You-Go (PAYG), Free Tier, EA/MCA, CSP, Dev/Test pricing; Consumption/Serverless vs. Provisioned capacity; **1-Year / 3-Year Reserved Instances** (up to ~72% off), **Azure Savings Plans for Compute** (up to ~65% off across compute SKUs/regions), **Spot VMs** (up to ~90% off with 30-second eviction notice), and **Azure Hybrid Benefit** (BYOL OS/SQL licensing discount).
- **5-Phase Cost Lifecycle:** (1) Discover & Scope client requirements (traffic pattern, SLA/RTO/RPO, commitment tolerance) $\rightarrow$ (2) Model & Tag (`Environment`, `Project`, `Owner`, `CostCenter`) $\rightarrow$ (3) Govern with Budgets, Action Groups, and Azure Policy SKU/location/tag guardrails $\rightarrow$ (4) Monitor with Cost Analysis & Anomaly Alerts $\rightarrow$ (5) Optimize via Azure Advisor, right-sizing, tiering, and commitments.
- **VM & Network Cost Optimization:**
  - Always **Deallocate** (`Stopped (deallocated)`) rather than shutting down inside the OS; schedule off-hours shutdowns (e.g., non-24/7 QA/dev fleets) and clean up unattached disks, snapshots, and unused Public IPs.
  - Right-size SKUs, consider burstable (`B-series`) or ARM Linux (`Dpsv5`/`Epsv5`) where suitable, and match disk tiers (Standard SSD vs. Premium SSD/Ultra Disk) to actual IOPS needs.
  - Keep internal traffic private within a VNet / same region and use **Private Endpoints** selectively where public egress or security justifies the endpoint + per-GB cost vs. public internet egress fees.

## Quick Revision Checklist
- [ ] Explain why `Stopped (deallocated)` differs from guest-OS shutdown in billing.
- [ ] Compare Reserved Instances vs. Savings Plans vs. Spot VMs vs. Azure Hybrid Benefit.
- [ ] Contrast Availability Sets (FDs/UDs) with VMSS (Flexible vs. Uniform, autoscale, scale-in).
- [ ] Decide between Shared RDP, Dedicated VM, and AVD for a given workload and SLA.
