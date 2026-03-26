# enterprise-server-revival
End-to-end hardware revival of an IBM x3250 M3. Features BIOS ROM recovery via IMM, hardware RAID bypass engineering, and 

Proxmox/ZFS virtualization deployment.
# 🛠️ enterprise-server-revival
### IBM System x3250 M3 (Type 4252) | SN#KQ1Y4KK

**Description:** End-to-end hardware revival of a decommissioned IBM 1U rack server. This project documents deep-level hardware diagnostics, out-of-band management (IMM) firmware recovery, proprietary RAID controller bypass engineering, and the deployment of a modern Proxmox/ZFS virtualization stack.

---

## 📊 System Architecture & Specifications

| Component | Post-Restoration State | Notes |
| :--- | :--- | :--- |
| **Compute** | Intel Xeon X3460 (4C/8T @ 2.80GHz) | Thermal interface fully repasted. |
| **Memory** | 32GB (4x8GB) PC3L-10600R RDIMM | Currently staged on 4GB diagnostic baseline. |
| **Storage Array** | 4x 120GB Micron Enterprise SSDs | Direct-Attached Storage (DAS) configuration. |
| **Filesystem** | ZFS RAID10 (Mirrored vdevs) | Capable of 1.3 GB/s sustained write speeds. |
| **Hypervisor** | Proxmox VE 9.1-1 | Configured with automated backup jobs. |
| **Management** | IBM Integrated Management Module | System Health: 100% Normal. |

---

## 🛠️ Phase 1: The "Salvage" & Decontamination
The unit was initially recovered in a heavily weathered state, requiring a full physical teardown to prevent electrical shorts or thermal throttling during testing.

* **Initial Audit:** Stripped the chassis to the bare motherboard. 
* **Precision Cleaning:** Decontaminated the primary fan bank, DIMM slots, and CPU socket area, re-establishing a safe baseline for electrical testing.

> **Gallery: Teardown & Cleaning**
> ![Initial State]
> https://via.placeholder.com/400x250?text=https://github.com/user-attachments/assets/4c7be348-8026-44cf-8b53-d5995085cfbe)

> ![Clean Motherboard]
> https://github.com/user-attachments/assets/347d5c98-d5d5-405b-bb98-5be59f2ce2c0

---

## ⚡ Phase 2: Electrical & OOB Diagnostics
The server was trapped in a "No-POST" state. Instead of replacing parts blindly, I utilized enterprise diagnostic methodologies to isolate the fault.

* **Electrical Benchmarking:** Utilized a digital multimeter to verify the PSU rails. Confirmed a stable **5.16V** delivery to the logic board, ruling out a power supply failure.
* **IMM Verification:** Accessed the out-of-band Integrated Management Module (IMM) to pull the system event logs. Discovered the root cause:
  > `Critical Error: Firmware BIOS (ROM) corruption detected on system SN#KQ1Y4KK during POST`

> **Gallery: Diagnostics**
> ![Multimeter Testing](https://via.placeholder.com/400x250?text=Insert+IMG_7686.MOV+Screenshot) ![IMM Interface](https://via.placeholder.com/400x250?text=Insert+IMG_7703.MOV+Screenshot)

---

## 🧠 Phase 3: Firmware Recovery & Logic Stabilization
The BIOS corruption caused severe interoperability hardware loops, resulting in a parasitic drain that killed three successive CMOS batteries.

* **Manual Logic Reset:** Manipulated the **CLR CMOS** and **BIOS Recovery** headers on the motherboard to force the system to initialize from the backup BIOS image.
* **Heartbeat Stabilization:** Successfully cleared the corruption. Diagnostic LEDs transitioned from amber faults to a solid green **STB PWR**, confirming the logic board was retaining settings and ready for POST.

> **Gallery: Firmware Recovery**
> ![Motherboard Jumpers](https://via.placeholder.com/400x250?text=Insert+IMG_7685.jpg) ![Green Status LEDs](https://via.placeholder.com/400x250?text=Insert+IMG_7688.MOV+Screenshot)

---

## 🔀 Phase 4: Storage Engineering & The Hardware Bypass
During initial configuration, the legacy LSI RAID controller suffered a critical failure and could no longer initialize the drive backplane. 

* **The Motherboard Bypass:** Rather than sourcing legacy replacement parts, I reverse-engineered the storage paths and rerouted the SAS/SATA backplane directly to the motherboard's onboard SATA ports.
* **Direct-Attached Storage (DAS):** This engineering workaround completely bypassed the failed hardware RAID, allowing the hypervisor to access the raw drives directly—an ideal scenario for modern software-defined storage.

> **Gallery: The Storage Bypass**
> ![Rerouting Cables](https://via.placeholder.com/400x250?text=Insert+IMG_7679.MOV+Screenshot) ![Micron SSD Array](https://via.placeholder.com/400x250?text=Insert+IMG_7756.jpg)

---

## 🚀 Phase 5: Virtualization & Performance Validation
With the hardware stabilized and the storage path cleared, the system was provisioned for modern enterprise workloads.

* **High-Speed ZFS:** Capitalizing on the bypass, Proxmox was allowed to manage the Micron SSDs directly via a ZFS RAID10 pool. Stress testing (`dd`) verified a sustained write speed of **1.3 GB/s**.
* **Service Deployment:** Successfully deployed an **Ubuntu 24.04 LTS** virtual machine and an **Nginx** containerized web server.
* **Automated Reliability:** Configured the environment for high availability, including "Start-on-Boot" parameters and 100% successful automated backup jobs.

> **Gallery: Hypervisor Performance**
> ![ZFS Speed Test](https://via.placeholder.com/400x250?text=Insert+zfs+speed+test.png) ![Proxmox Backups](https://via.placeholder.com/400x250?text=Insert+successful+backup.png)

---

## 🏆 Phase 6: Final Showroom State
The transition from a decommissioned, non-posting unit to a surgically clean, high-performance virtualization host is complete. The system boasts organized cable management, optimized airflow, and 100% verified hardware health.

> **Gallery: Ready for Deployment**
> ![Final Build Interior](https://via.placeholder.com/400x250?text=Insert+IMG_7762.MOV+Screenshot) ![Front Bezel Online](https://via.placeholder.com/400x250?text=Insert+IMG_7777.JPEG)

---
*Project maintained by [Your GitHub Username / JPARO Gadgets]*
