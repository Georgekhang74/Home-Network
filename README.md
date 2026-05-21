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

### Trunk Port (to AP)

- Native VLAN: 999  
- Allowed VLANs: 20,30,70,999  


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

