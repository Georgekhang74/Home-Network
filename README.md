# Home-Network
Creating my home network
# Home Network Rebuild – pfSense + Cisco + Omada SDN

This repository documents the full rebuild of my home network, including:

- pfSense firewall/router
- Cisco 2960X (trunking + VLANs)
- TP-Link Omada EAP access point
- Multi-VLAN Wi-Fi design (Personal, IoT, Guest)
- Network segmentation and security rules

---

## 📌 Network Topology Overview


                    ┌────────────────────┐
                    │      Internet      │
                    └────────┬───────────┘
                             │
                             │ WAN
                     ┌───────┴────────┐
                     │    pfSense     │
                     │ Firewall/Router│
                     │                │
             VLAN 70 → 10.0.70.1/24   │
             VLAN 20 → 10.0.20.1/24   │
             VLAN 30 → 10.0.30.1/24   │
                     └───────┬────────┘
                             │
                             │ Trunk (VLANs 20,30,70)
                     ┌───────┴────────┐
                     │  Cisco Switch  │
                     └───────┬────────┘
                             │
                             │ Trunk (VLANs 20,30,70)
                     ┌───────┴────────┐
                     │   Omada EAP    │
                     │   Wi-Fi AP     │
                     └───────┬────────┘
                 ┌───────────┼───────────────────────────┐
                 │           │                           │    
    SSID: Chicken(Main)  SSID: Flamingo(IOT)    SSID: Birds(Guest)
           VLAN 70            VLAN 20                VLAN 30
          (Personal)          (IoT)                  (Guest)


---

## 🧩 VLAN Design

| VLAN | Purpose | Subnet | Notes |
|------|---------|---------|-------|
| 70 | Personal LAN | 10.0.70.0/24 | Main trusted network |
| 20 | IoT | 10.0.20.0/24 | Restricted devices |
| 30 | Guest | 10.0.30.0/24 | Internet-only |
| 10 | LAN Port| N/A | Used for pfsense |


---

## 🔧 pfSense Configuration

### 1. VLAN Interfaces

- VLAN 70 → Personal LAN  
- VLAN 20 → IoT  
- VLAN 30 → Guest  
- VLAN 999 → Native/Trunk (no IP)


---

### 2. DHCP Configuration

Each VLAN has its own DHCP scope:

- VLAN 70: 10.0.70.50–10.0.70.199  
- VLAN 20: 10.0.20.50–10.0.20.199  
- VLAN 30: 10.0.30.50–10.0.30.199  


---

### 3. Firewall Rules

Each VLAN is isolated with rules such as:

- Allow VLAN → WAN  
- Block VLAN → LAN  
- Block VLAN → other VLANs  
- Allow DNS/DHCP to pfSense  


---

## 🛰 Cisco Switch Configuration


- Native VLAN: 10 
- Allowed VLANs: 10,20,30,70,999  
<img width="642" height="596" alt="Screenshot 2026-05-21 161457" src="https://github.com/user-attachments/assets/e4888a42-d526-427c-b012-ea8292095ec8" />
<img width="485" height="244" alt="Screenshot 2026-05-21 161339" src="https://github.com/user-attachments/assets/912392a0-3002-4916-ba5f-93cbdbdf4263" />

- Links to the router and AP are trunks allowing the selected vlan traffic through


- Ignore VLAN 999, I was going to use it as a blackhole vlan but had connectivity issues so I decided just to leave it there for now. 
- Vlan 10 exists for connectivity to my pfsense router

<img width="368" height="92" alt="Screenshot 2026-05-19 190741" src="https://github.com/user-attachments/assets/186ad119-ebd7-4172-bb78-33aef08f015e" />

- I configured a switchport to allow myself into the pfsense web ui
  - This is redundant because I can get in with multiple other IP addresses
---

## 📡 Omada Controller Configuration

### SSIDs

| SSID | VLAN | Purpose |
|------|------|----------|
| Chicken=p | 70 | Personal LAN |
| Flamingo-i | 20 | IoT devices |
| Birds | 30 | Guest network |


---

### SSID Advanced Settings (VLAN Tagging)

Each SSID has VLAN tagging configured under:

**WLAN → Edit → Advanced → VLAN**


---

## 🧪 Testing & Verification

### 1. IP Address Assignment  
Connect to each SSID and verify the correct subnet:

- Main → 10.0.70.x  
- IoT → 10.0.20.x  
- Guest → 10.0.30.x  


---

### 2. Switch MAC Table Verification

Use:


---

## 🏁 Final Notes

This repo documents the full rebuild of my home network using enterprise-style segmentation, VLANs, and controller-based Wi-Fi.  
Future updates may include:

- VLAN optimization
- Network clean-up
- Home servers
- VPN integration  
- ZeroTier or WireGuard  
- Additional APs  
- Monitoring dashboards  

