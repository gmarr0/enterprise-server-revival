# 🛠️ enterprise-server-revival

### IBM System x3250 M3 (Type 4252) | SN#KQ1Y4KK

**Description:** End-to-end hardware revival of a decommissioned IBM 1U rack server. This project documents deep-level hardware diagnostics, out-of-band management (IMM) firmware recovery, proprietary RAID controller bypass engineering, memory compatibility troubleshooting, and the deployment of a modern Proxmox/ZFS virtualization stack.

---

## 📊 System Architecture & Specifications

| Component | Post-Restoration State | Notes |
|---|---|---|
| **Compute** | Intel Xeon X3460 (4C/8T @ 2.80GHz) | Thermal interface fully repasted. |
| **Memory** | 12GB (3x4GB) PC3-10600U UDIMM ECC | Slots 1, 2 & 4 populated. Slot 5 identified as non-functional via isolation testing. |
| **Storage Array** | 4x 120GB Micron Enterprise SSDs | Direct-Attached Storage (DAS) configuration. |
| **Filesystem** | ZFS RAID10 (Mirrored vdevs) | Verified 1.3 GB/s sustained write speeds. |
| **Hypervisor** | Proxmox VE 9.1-1 | Configured with automated backup jobs. |
| **Firmware** | IBM uEFI v1.19 (latest available) | Updated via IMM remote management interface. |
| **Management** | IBM Integrated Management Module (IMM) | System Health: 100% Normal. |

---

## 🛠️ Phase 1: The "Salvage" & Decontamination

The unit was initially recovered in a heavily weathered state, requiring a full physical teardown to prevent electrical shorts or thermal throttling during testing.

* **Initial Audit:** Stripped the chassis to the bare motherboard.
* **Precision Cleaning:** Decontaminated the primary fan bank, DIMM slots, and CPU socket area, re-establishing a safe baseline for electrical testing.

> **Gallery: Teardown & Cleaning**
>
> [Initial State]

<img width="703" height="935" alt="dirty server" src="https://github.com/user-attachments/assets/1395071b-f408-4cae-938a-65c350a3d6e4" />

> [IMM Boot Diagnostics]

<img width="703" height="935" alt="dirty server" src="https://github.com/user-attachments/assets/46a14d4a-53f9-41e7-bddd-961de4932caa" />

---

## ⚡ Phase 2: Electrical & OOB Diagnostics

The server was trapped in a "No-POST" state. Instead of replacing parts blindly, I utilized enterprise diagnostic methodologies to isolate the fault.

* **Electrical Benchmarking:** Utilized a digital multimeter to verify the PSU rails. Confirmed a stable **5.16V** delivery to the logic board, ruling out a power supply failure.
* **IMM Verification:** Accessed the out-of-band Integrated Management Module (IMM) to pull the system event logs and identify the root cause.
* **Memory Slot Diagnostics:** Through systematic stick-by-stick isolation testing, identified **slot 5 as non-functional** — a hardware-level fault confirmed independently of RAM brand or spec. This was discovered during the RAM upgrade phase and documented as a known hardware limitation.

> **Gallery: Diagnostics**
>
> [Multimeter Testing]

<img width="703" height="935" alt="dirty server" src="https://github.com/user-attachments/assets/387642d0-dbfe-454f-91db-0785086d6f46" />"

---

## 🧠 Phase 3: Firmware Recovery & Logic Stabilization

The BIOS corruption caused severe interoperability hardware loops, resulting in a parasitic drain that killed three successive CMOS batteries.

* **Manual Logic Reset:** Manipulated the **CLR CMOS** and **BIOS Recovery** headers on the motherboard to force the system to initialize from the backup BIOS image.
* **Heartbeat Stabilization:** Successfully cleared the corruption. Diagnostic LEDs transitioned from amber faults to a solid green **STB PWR**, confirming the logic board was retaining settings and ready for POST.
* **Firmware Update via IMM:** With the system stabilized, flashed the uEFI firmware from an outdated version to **v1.19** (the latest available release) via the IMM remote management dashboard — resolving a known "uEFI Starting" POST hang that affects this platform under new memory configurations. IBM's original FTP download servers for this firmware have been decommissioned; sourcing the update required locating a verified third-party mirror.

> **Gallery: Firmware Recovery**
>
> [Motherboard Jumpers]

<img width="703" height="935" alt="Motherboard Jumpers" src="https://github.com/user-attachments/assets/6a424dea-db03-4859-b04f-74b797fbcb29" />


> [Green Status LEDs]

<img width="703" height="935" alt="Green Status LEDs" src="https://github.com/user-attachments/assets/cdce9a4d-31bb-45eb-a3b9-c62120c58e2e" />

> [IMM Interface — Server Operating Normally]

<img width="703" height="935" alt="Imm Interface server operating normally" src="https://github.com/user-attachments/assets/a3c06297-147a-48ae-96e6-4f2d78f4cbaf" />

---

## 💾 Phase 4: Memory Compatibility Engineering

Upgrading the RAM on legacy IBM server hardware is significantly more complex than it appears on the surface — and this phase involved substantial real-world debugging to resolve.

* **The RDIMM Trap:** An attempt to upgrade to 32GB using PC3-10600R RDIMMs failed completely — the system would not POST. Investigation revealed that while the x3250 M3 motherboard physically accepts RDIMM slots, the **Xeon X3460 CPU does not support registered memory at the controller level**. IBM's spec sheets list RDIMM capacity for other supported CPUs, making this a well-known source of confusion. The correct memory type for this CPU is **ECC Unbuffered (UDIMM)**, identifiable by the "U" or "E" suffix on the part number.
* **Slot Fault Isolation:** After sourcing compatible PC3-10600U UDIMMs, a "uEFI Starting" hang persisted with 4 sticks installed. Systematic isolation — testing each stick individually and in every slot combination — identified **slot 5 as non-functional**, independent of which stick was installed. The final working configuration uses slots 1, 2, and 4 for a total of 12GB.
* **Firmware Dependency:** The "uEFI Starting" hang is a documented bug in early firmware versions for this platform. Updating to v1.19 (see Phase 3) was required before the system would reliably POST with the new memory configuration.

---

## 🔀 Phase 5: Storage Engineering & The Hardware Bypass

During initial configuration, the legacy LSI RAID controller suffered a critical failure and could no longer initialize the drive backplane.

* **The Motherboard Bypass:** Rather than sourcing legacy replacement parts, I reverse-engineered the storage paths and rerouted the SAS/SATA backplane directly to the motherboard's onboard SATA ports.
* **Direct-Attached Storage (DAS):** This engineering workaround completely bypassed the failed hardware RAID, allowing the hypervisor to access the raw drives directly — an ideal scenario for modern software-defined storage.

> **Gallery: The Storage Bypass**
>
> [Rerouting Cables]

<img width="703" height="935" alt="Rerouting Cables" src="https://github.com/user-attachments/assets/1790aaff-cd58-4bc9-acf1-7d6e797f0cdc" />

> [Micron SSD Array]

<img width="703" height="935" alt="Micron SSD Array" src="https://github.com/user-attachments/assets/8c01cf83-2ce0-48dc-858f-875e6cbb2147" />

---

## 🚀 Phase 6: Virtualization & Performance Validation

With the hardware stabilized and the storage path cleared, the system was provisioned for modern enterprise workloads.

* **High-Speed ZFS:** Capitalizing on the bypass, Proxmox was allowed to manage the Micron SSDs directly via a ZFS RAID10 pool. Stress testing (`dd`) verified a sustained write speed of **1.3 GB/s**.
* **Service Deployment:** Successfully deployed an **Ubuntu 24.04 LTS** virtual machine and an **Nginx** containerized web server as live demo environments.
* **Automated Reliability:** Configured the environment for high availability, including "Start-on-Boot" parameters and 100% successful automated backup jobs.

> **Gallery: Hypervisor Performance**
>
> [ZFS Speed Test]

<img width="1920" height="1032" alt="zfs speed test" src="https://github.com/user-attachments/assets/8b35e9ae-b9e7-40f9-b2e1-e1e7d2d863f1" />

> [Proxmox Backups]

<img width="1920" height="1032" alt="successful backup" src="https://github.com/user-attachments/assets/8df53493-7b34-4562-9a7c-60e4e091604a" />
---

## 🏆 Phase 7: Final Showroom State

The transition from a decommissioned, non-posting unit to a surgically clean, high-performance virtualization host is complete. The system boasts organized cable management, optimized airflow, and 100% verified hardware health.

> **Gallery: Ready for Deployment**
>
> [Final Build Interior]

<img width="703" height="935" alt="Clean Motherboard" src="https://github.com/user-attachments/assets/5debe152-abb8-42ab-834a-d06202be4be1" />

> [Front Bezel Online]

<img width="703" height="935" alt="Front Bezel Online" src="https://github.com/user-attachments/assets/3afc78ca-45af-48d8-a67f-a4df840f2282" />
---

*Project maintained by [gmarr0](https://github.com/gmarr0)*
