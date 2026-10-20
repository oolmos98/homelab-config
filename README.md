# 📝 INTEGRATED HOME LAB & HOMEKIT MIGRATION BLUEPRINT

## 🚀 Project Objective
Migrate an expanding smart home layout from cloud-dependent silos into **Home Assistant OS (HAOS)** running on a dedicated **GMKtec M8 Mini PC (AMD Ryzen 5 PRO 6650H)** backed by a **UniFi Express 7 (UX7)** network router on **AT&T Fiber/Broadband**. The primary objective is to route all multi-brand hardware (**Ring Alarm & Doorbell, Wyze Cam v4, Costco Night Owl 12-Channel DVR, Meross smart plugs/garage opener, and Rachio 3 sprinkler controller**) into Home Assistant, bridge them seamlessly into **Apple HomeKit** for unified Siri and iOS Home App control, and establish a local-first networking and **Frigate NVR** video foundation.

---

## 🛠 Target Hardware & Infrastructure Profile

* **Host Server (The Mini PC):** GMKtec M8 Mini PC
  * **Specifications:** AMD Ryzen 5 PRO 6650H processor (with Radeon 660M iGPU), 16GB LPDDR5 RAM, and a 1TB SSD.
  * **Networking Perks:** Equipped with Dual 2.5GbE LAN network interface ports, providing massive physical bandwidth for separating smart home traffic from internal server data routing.
* **Networking Backbone:** UniFi Express 7 (UX7)
  * **Role:** High-end Wi-Fi 7 Cloud Gateway serving as the primary network router and firewall.
  * **ISP Transition:** Planned deployment alongside a new AT&T Internet fiber or broadband modem/gateway.
* **Video & Surveillance Footprint:**
  * **Existing Cloud:** Wyze Cam v4 and a Ring Video Doorbell.
  * **Local Analog Expansion:** Night Owl 12-Channel DVR Security System from Costco (featuring 6 wired 2K analog BNC cameras and a local 1TB storage drive).
* **Peripherals & Core Smart Utilities:**
  * **Existing Outlets:** Meross Wi-Fi wall plugs.
  * **Garage Access:** Meross Smart Garage Door Opener Remote (MSG100HK).
  * **Irrigation:** Rachio 3 Smart Sprinkler Irrigation Controller (8-Zone smart home ready ecosystem model).
* **User Environment Constraints:** Apple-centric ecosystem (iPhone, Mac computer layout, Apple TV setup), driving a requirement for direct Apple HomeKit exposure.

---

## 🗺 Advanced Integration Map

| Device Platform | Home Assistant Ingestion Method | Apple HomeKit Exposure Style |
| :--- | :--- | :--- |
| **Ring Alarm & Doorbell** | Ring-MQTT Add-on + Mosquitto Broker | Exposes raw contact/motion zones. Security system panel populates as a native armed security dashboard layout. |
| **Wyze Cam v4** | Docker Wyze Bridge Add-on | Converts cloud stream to local RTSP feed for real-time video tile display. |
| **Night Owl DVR (Costco)** | Local RTSP over port 554 (`/cam/realmonitor?channel=X&subtype=0`) via Generic Camera integration | Passes local high-res video feeds straight to Apple devices, entirely bypassing the Night Owl cloud framework. |
| **Meross Plugs & Garage** | Native Meross LAN/Cloud Integration | Plugs display as immediate switches; the MSG100HK presents as a native secure Garage Door module with CarPlay display prompts. |
| **Rachio 3 Sprinkler** | Built-in Rachio Smart Home integration | Exposes watering zone configurations, allowing local automation overrides or smart voice controls via Siri. |

---

## 📦 Phase 1: Mini PC & Network Provisioning Blueprint

1. **HAOS Installation Matrix:** Use BalenaEtcher to flash the official Generic x86-64 HAOS image to an external USB stick. Boot the GMKtec M8 into BIOS, disable Secure Boot, and configure the motherboard power state to **Restore on AC Power Loss** (guaranteeing auto-booting following an outage).
2. **AT&T Gateway to UniFi Configuration:** To ensure the UniFi Express 7 functions as the primary edge firewall and router without creating a restrictive Double-NAT configuration, the AT&T residential gateway must be set to **IP Passthrough Mode** (with its internal Wi-Fi radios disabled completely).
3. **Local Network Isolation Strategy:** Leverage one of the GMKtec PC's dual 2.5GbE network interface cards to wire directly into the UX7 LAN network. Set a **Static Local IP Address** inside Home Assistant to guarantee dashboard access continuity.

---

## 📈 Multi-Service Docker Co-Hosting Goals

Using the x86-64 processing headroom of the Ryzen 5 PRO 6650H chip, the following microservices are queued for companion hosting inside the Home Assistant OS infrastructure:
* **AdGuard Home:** Local DNS sinkhole for system-wide tracker and ad mitigation.
* **Frigate NVR:** Onboard graphics acceleration (Radeon 660M) to run local AI object/human parsing against the local Night Owl and Wyze RTSP camera feeds.
* **Tailscale / WireGuard:** Encrypted endpoint proxy for secure out-of-home panel controls without opening router ports.
