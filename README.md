# Home-Network
Creating my home network
# Home Network Rebuild – pfSense + Cisco + Omada SDN

This repository documents the rebuild of my home network, including:

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


<img width="4032" height="3024" alt="IMG_8942" src="https://github.com/user-attachments/assets/300b8d2b-96e0-4f15-8ac9-8d10f372326e" />

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
- VLAN 10 → Native/Management (10.0.10.0/24)

<img width="1429" height="254" alt="Screenshot 2026-05-21 160128" src="https://github.com/user-attachments/assets/a97f9e0d-cf75-4476-a1ad-21fc9110dee2" />
<br><br>
<img width="1348" height="688" alt="image" src="https://github.com/user-attachments/assets/7c6aaab1-0303-4806-b110-9c0868ce2f70" />

- VLAN 10 is not shown in pfsense because it is the "LAN" port



---

### 2. DHCP Configuration

Each VLAN has its own DHCP scope:

- VLAN 70: 10.0.70.50–10.0.70.245  
- VLAN 20: 10.0.20.50–10.0.20.199  
- VLAN 30: 10.0.30.50–10.0.30.199
- VLAN 10: 10.0.10.10–10.0.10.245
<img width="1580" height="835" alt="image" src="https://github.com/user-attachments/assets/a81152b9-ba36-4dfa-be30-8822eea47bc5" />

<br><br>

<img width="1518" height="819" alt="image" src="https://github.com/user-attachments/assets/2ad9e54f-08f1-416f-ac6e-2f47b687d07a" />

<br><br>

<img width="1522" height="819" alt="image" src="https://github.com/user-attachments/assets/6c7b78ac-dc78-4f09-8ac2-7ba0d53d83f1" />

<br><br>

<img width="1435" height="830" alt="image" src="https://github.com/user-attachments/assets/708e6f18-54ee-4fe2-ae63-b875364ebfe3" />



---

### 3. Firewall Rules

Each VLAN is isolated with rules such as:

- Allow VLAN → WAN  
- Block VLAN → LAN  
- Block VLAN → other VLANs  
- Allow DNS/DHCP to pfSense  
<img width="1453" height="583" alt="image" src="https://github.com/user-attachments/assets/bd712fbf-81e5-42fa-8973-08b81c76da73" />

&nbsp;

<img width="1493" height="344" alt="image" src="https://github.com/user-attachments/assets/b85e4197-9e9e-402c-b2dc-074e81f3142b" />
<br><br>
<img width="1479" height="242" alt="image" src="https://github.com/user-attachments/assets/c008963c-4cca-4e01-80ce-98bc9969c816" />

<br><br>

<img width="1463" height="301" alt="image" src="https://github.com/user-attachments/assets/cfb6885e-c67d-4404-a225-eab85909c7b1" />

<br><br>

---

## 🛰 Cisco Switch Configuration


- Native VLAN: 10 
- Allowed VLANs: 10,20,30,70,999  
<img width="642" height="596" alt="Screenshot 2026-05-21 161457" src="https://github.com/user-attachments/assets/e4888a42-d526-427c-b012-ea8292095ec8" />
<br><br>
<img width="485" height="244" alt="Screenshot 2026-05-21 161339" src="https://github.com/user-attachments/assets/912392a0-3002-4916-ba5f-93cbdbdf4263" />

- Links to the router and AP are trunks allowing the selected vlan traffic through


- Ignore VLAN 999, I was going to use it as a blackhole vlan but had connectivity issues so I decided just to leave it there for now. 
- Vlan 10 exists for connectivity to my pfsense router

<img width="368" height="92" alt="Screenshot 2026-05-19 190741" src="https://github.com/user-attachments/assets/186ad119-ebd7-4172-bb78-33aef08f015e" />
<br><br>

- I configured a switchport to allow myself into the pfsense web ui
  - This is redundant because I can get in with multiple other IP addresses however, I wanted a managent port.
---

## 📡 Omada Controller Configuration

### SSIDs

| SSID | VLAN | Purpose |
|------|------|----------|
| Chicken-p | 70 | Personal LAN |
| Flamingo-i | 20 | IoT devices |
| Birds | 30 | Guest network |
<img width="1557" height="407" alt="image" src="https://github.com/user-attachments/assets/5d916927-67bc-48c1-88a7-bfbf762d4ce7" />

Each SSID has VLAN tagging configured under:

**WLAN → Edit → Advanced → VLAN**

---



## 🧪 Testing & Verification

### 1. IP Address Assignment  
Connect to each SSID and verify the correct subnet:

- Main(Personal) → 10.0.70.x  
- IoT → 10.0.20.x  
- Guest → 10.0.30.x  
<img width="286" height="251" alt="Screenshot 2026-05-21 153822" src="https://github.com/user-attachments/assets/b1e36496-0103-4fb8-ad36-807b5b576f09" />
<br><br>
<img width="892" height="187" alt="Screenshot 2026-05-21 161706" src="https://github.com/user-attachments/assets/493956b8-0d4a-473b-84ed-b72a530362c9" />
<br><br>
<img width="825" height="181" alt="Screenshot 2026-05-21 154039" src="https://github.com/user-attachments/assets/93c6c358-f950-4a91-be99-3aaff053c37e" />
<br><br>
<img width="908" height="171" alt="Screenshot 2026-05-21 154152" src="https://github.com/user-attachments/assets/ac9fe56e-9057-404f-b774-8a2548d5189a" />


---


## 🏁 Final Notes


This is the finished version of my home network. If you are wondering why I have a random vlan... you are free to check out my more detailed and documented version of this lab, where I show where I went wrong and the places I had to redesign/reconfigure. 

https://github.com/Georgekhang74/Home-Network-The-Process-

---

This repo documents the full rebuild of my home network using enterprise-style segmentation, VLANs, and controller-based Wi-Fi.  
Future updates may include:

- VLAN optimization
- Network clean-up
- Home servers
- VPN integration  
- ZeroTier or WireGuard  
- Additional APs  
- Monitoring dashboards  



