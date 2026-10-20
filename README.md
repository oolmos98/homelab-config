# 📝 INTEGRATED HOME LAB & HOMEKIT MIGRATION BLUEPRINT (v3.0)

## 🚀 Project Objective
Migrate an expanding smart home layout from cloud-dependent silos into a virtualized **Proxmox VE** cluster running **Home Assistant OS (HAOS)**, **Frigate NVR**, and **Scrypted** on a dedicated **GMKtec M8 Mini PC (AMD Ryzen 5 PRO 6650H)** backed by a **UniFi Express 7 (UX7)** router and a **UniFi Flex Mini 2.5G Switch** on **AT&T Fiber/Broadband**. The primary objective is to route all multi-brand hardware (**Ring Alarm & Doorbell, Wyze Cam v4, Costco Night Owl 12-Channel DVR, Meross smart plugs/garage opener, and Rachio 3 sprinkler controller**) into Home Assistant, bridge them seamlessly into **Apple HomeKit** (with **HomeKit Secure Video**) for unified Siri, CarPlay, and iOS Home App control, and establish a fault-isolated, local-first networking and video foundation.

---

## 🛠 Target Hardware & Infrastructure Profile

* **Host Hypervisor Server:** GMKtec M8 Mini PC (Running **Proxmox VE**)
  * **Specifications:** AMD Ryzen 5 PRO 6650H processor (6C/12T with Radeon 660M iGPU), 16GB LPDDR5 RAM, and a 1TB NVMe SSD.
  * **Networking Perks:** Dual 2.5GbE LAN NICs mapped to Proxmox virtual bridges (`vmbr0` for main LAN traffic; `vmbr1` optional dedicated Camera/IoT ingestion trunk).
* **Networking Backbone:** UniFi Express 7 (UX7) + UniFi Flex Mini 2.5G Switch
  * **Router Role:** High-end Wi-Fi 7 Cloud Gateway serving as the primary network router, DHCP server, and edge firewall.
  * **Switching Expansion:** **UniFi Flex Mini 2.5G** (5-Port 2.5GbE Managed Switch) uplinked to the single UX7 LAN port to hardwire the GMKtec M8, Night Owl DVR, and Apple TV at full 2.5GbE speeds.
  * **ISP Transition & Strict Wiring Rule:** AT&T Fiber/Broadband Gateway set to **IP Passthrough Mode** with Wi-Fi disabled. **ONLY ONE Ethernet cable** connects the AT&T Gateway to the UX7 WAN port. All other AT&T Gateway Ethernet ports remain completely empty to prevent Split-NAT and mDNS/HomeKit isolation.
* **Video & Surveillance Footprint:**
  * **Existing Cloud:** Wyze Cam v4 and a Ring Video Doorbell.
  * **Local Analog Expansion:** Night Owl 12-Channel DVR Security System from Costco (6 wired 2K analog BNC cameras + 1TB local storage drive).
* **Peripherals & Core Smart Utilities:**
  * **Existing Outlets:** Meross Wi-Fi wall plugs.
  * **Garage Access:** Meross Smart Garage Door Opener Remote (MSG100HK).
  * **Irrigation:** Rachio 3 Smart Sprinkler Irrigation Controller (8-Zone).
* **User Environment Constraints:** Apple-centric ecosystem (iPhone, Mac, Apple TV Home Hub), requiring direct Apple HomeKit and HomeKit Secure Video (HKSV) exposure.

---

## 🖥 Proxmox VE Virtualization Layout (GMKtec M8)

| ID | Type | Name | Resources | Role & Hardware Passthrough |
| :--- | :--- | :--- | :--- | :--- |
| **VM 100** | KVM VM | **Home Assistant OS** | 2 vCPU / 4GB RAM / 64GB Disk | Full HAOS appliance with Add-on Store, HACS, Ring-MQTT, Mosquitto, and Meross LAN. Snapshot-enabled for 10-second update rollbacks. |
| **LXC 101** | Container | **Frigate NVR + Go2RTC** | 4 vCPU / 6GB RAM / 500GB Disk | Local 24/7 NVR & AI object detection. Direct LXC passthrough of AMD Radeon 660M iGPU (`/dev/dri/renderD128`) for VAAPI/ROCm acceleration. |
| **LXC 102** | Container | **Scrypted** | 2 vCPU / 2GB RAM / 16GB Disk | Dedicated HomeKit Secure Video (HKSV) bridge for Night Owl (via Go2RTC), Wyze, and Ring Doorbell. Survives HAOS reboots without dropping cameras. |
| **LXC 103** | Container | **AdGuard Home + Tailscale** | 1 vCPU / 1GB RAM / 8GB Disk | Network-wide DNS sinkhole and encrypted subnet router VPN. Isolated from HAOS so household DNS and remote Proxmox recovery stay up 24/7. |

---

## 🗺 Advanced Integration Map

| Device Platform | Ingestion Method | Apple HomeKit Exposure Style |
| :--- | :--- | :--- |
| **Ring Alarm & Doorbell** | Ring-MQTT Add-on (HAOS VM 100) + Scrypted Ring Plugin (LXC 102) | Alarm exposes contact/motion zones & native armed dashboard. Doorbell triggers Apple TV PIP popups and HKSV chime alerts. |
| **Wyze Cam v4** | Docker Wyze Bridge -> Frigate (LXC 101) -> Scrypted (LXC 102) | Converts cloud stream to local RTSP feed, exposed to HomeKit with HKSV iCloud recording and motion zones. |
| **Night Owl DVR (Costco)** | Local RTSP over port 554 (`/cam/realmonitor?channel=X&subtype=0`) -> Frigate Go2RTC (LXC 101) -> Scrypted (LXC 102) | Passes low-latency local streams straight to Apple devices with full HKSV support, bypassing Night Owl cloud. |
| **Meross Plugs & Garage** | **Meross LAN (HACS)** / HomeKit Device Local Integration (VM 100) | Ingested locally into HA first, then re-exported via HomeKit Bridge. MSG100HK triggers native CarPlay garage prompts with zero cloud dependency. |
| **Rachio 3 Sprinkler** | Built-in Rachio Integration (VM 100) | Exposes watering zone configurations, allowing local automation overrides or Siri voice controls. |

---

## 📦 Phase 1: Provisioning Execution Sequence

1. **AT&T Gateway & UniFi Backbone Setup:**
   * Disable 2.4GHz and 5GHz Wi-Fi on the AT&T Gateway and enable **IP Passthrough (DHCPS-fixed)** pointing to the UniFi Express 7 WAN MAC address.
   * Run a single cable from the AT&T Gateway's primary port (Blue 5GbE port if BGW320) to the UX7 WAN port. Leave all other AT&T ports empty.
   * Connect the **UniFi Flex Mini 2.5G Switch** to the UX7 LAN port and wire the GMKtec M8, Night Owl DVR, and Apple TV into the switch.
2. **GMKtec M8 BIOS & Proxmox VE Installation:**
   * Boot GMKtec M8 into BIOS (`Del`/`Esc`), disable Secure Boot, enable SVM (AMD Virtualization) & IOMMU, and set AC Power Loss to **Power On / Restore**.
   * Flash **Proxmox VE ISO** via Ventoy or BalenaEtcher to USB and install onto the 1TB NVMe SSD with a static LAN IP.
3. **Rapid Stack Deployment (Proxmox Community Scripts):**
   * Execute Proxmox VE Community Helper Scripts in the Proxmox shell to automatically provision **VM 100 (HAOS)**, **LXC 101 (Frigate/Docker)**, **LXC 102 (Scrypted)**, and **LXC 103 (AdGuard + Tailscale)**.
4. **Firewall Lockdown:**
   * Assign static DHCP leases in UniFi for Proxmox, HAOS, Frigate, Scrypted, AdGuard, and the Night Owl DVR.
   * Create a UniFi Firewall Traffic Rule blocking the Night Owl DVR IP from **Internet (WAN) Access** while permitting local port 554 RTSP traffic to LXC 101 (Frigate).
