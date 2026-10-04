# ⚡ DHCP Starvation — Security Lab Demonstration

[![GitHub](https://img.shields.io/badge/github-repo-3776AB?style=for-the-badge&logo=github&logoColor=white)](https://github.com/)
[![Python](https://img.shields.io/badge/Python-3.x-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Scapy](https://img.shields.io/badge/Scapy-Network%20Packets-red?style=for-the-badge)](https://scapy.net/)
[![LAB](https://img.shields.io/badge/Security-Lab-8A2BE2?style=for-the-badge)]("https://github.com/3078756D627261/DHCPStarv/edit/main/README.md")

> ⚠️ For authorized security testing and isolated laboratories only.

This tool generates crafted DHCP traffic and can consume addresses from a DHCP server's address pool. Never use it against networks or systems without explicit authorization.

---

## 📖 Overview

DHCP starvation is a denial-of-service technique that attempts to exhaust the available addresses in a DHCP server's address pool.
This project demonstrates the underlying DHCP exchange using Python and Scapy. It generates DHCP traffic using locally administered MAC addresses and observes DHCP server responses.

The project is intended for:

- 🔬 Security research
- 🧪 Isolated penetration-testing labs
- 🎓 Network-security education
- 🛡️ Understanding DHCP-based attacks
- 🔍 Studying defensive controls such as DHCP snooping

---

## ⚙️ How It Works

At a high level, the program performs the following workflow:

```text
              ┌──────────────────────┐
              │   Test DHCP Server   │
              │                      │
              │   Address Pool       │
              └──────────┬───────────┘
                         │
              DHCP OFFER │
                         ▼
┌─────────────────────────────────────────────────┐
│                  Test Machine                   │
│                                                 │
│  Generate MAC ──► DHCP DISCOVER ──► Server      │
│       │                                         │
│       └────── Generate another MAC              │
│                         │                       │
│                         ▼                       │
│                  DHCP DISCOVER                  │
│                         │                       │
│                         ▼                       │
│                  DHCP REQUEST                   │
└─────────────────────────────────────────────────┘
```

The script:
1. Generates a locally administered MAC address.
2. Starts a DHCP packet sniffer in a background thread.
3. Creates a DHCP `DISCOVER` packet.
4. Sends the packet through the selected interface.
5. Monitors DHCP traffic for an `OFFER`.
6. Extracts information from the offer.
7. Builds a corresponding DHCP `REQUEST`.
8. Repeats the process for the configured test count.

---

## 🧰 Requirements
#### Software

- Python 3
- Scapy

Install Scapy with:

```text
python3 -m pip install scapy
sudo apt-get install python3-scapy
```

Verify the installation:

```text
python3 -c "from scapy.all import *; print('Scapy OK')"
```

### Network

For safe testing, use an isolated laboratory network containing:

- A test DHCP server
- A test client/machine
- A dedicated network interface or virtual network
- A deliberately small DHCP address pool

---

## 🚀 Usage

Clone the repository and enter the project directory:

```text
git clone https://github.com/3078756D627261/DHCPStarv.git
cd DHCPStarv
```

Display the available options:

```text
python3 DHCPStarv.py --help
```

#### Command-line options
| Option | Long form | Description | Default |
|---|---|---|---|
| -i |	--iface | Network interface | eth0 |
| -c |	--count | Number of DHCP DISCOVER packets | 10 |

For an authorized isolated lab, provide your test interface and a small packet count:

```text
python3 DHCPStarv.py -i <lab-interface> -c <small-test-count>
```

> 💡 Start with a very small test environment and packet count so you can observe the DHCP exchange without unnecessarily exhausting the test server's pool.

---

## 🧩 Project Structure

```text
.
├── DHCPStarv.py
└── README.md
```

---

## 🔬 Code Overview
`generate_random_mac()`

Generates a locally administered MAC address.

The generated address begins with `02`, which indicates that the MAC is locally administered.

```text
02:xx:xx:xx:xx:xx
```
---

`build_dhcp_discover()`

Constructs a DHCP DISCOVER packet using Scapy.

The packet contains:

```text
Ethernet
   │
   └── IPv4
         │
         └── UDP
               │
               └── BOOTP
                     │
                     └── DHCP DISCOVER
```
---

`packet_sniffer()`

Starts a packet capture on the selected interface and monitors DHCP traffic.

The capture filter is:

```text
udp and (port 67 or 68)
```

Captured packets are passed to `packet_handler()`.

---

`packet_handler()`

Processes captured DHCP packets.

When a DHCP `OFFER` is detected, the function extracts information including:

- Transaction ID
- Offered IP address
- DHCP server address

It then constructs a DHCP `REQUEST`.

---

`build_dhcp_request()`

Builds the DHCP `REQUEST` corresponding to a received `OFFER`.

The request contains the requested address and DHCP server identifier obtained from the offer.

---

`send_discover_packets()`

Responsible for transmitting the generated DHCP `DISCOVER` packet through the selected interface.

---

`main()`

Coordinates the application:

```text
Arguments
    │
    ▼
Generate sniffer MAC
    │
    ▼
Start sniffer thread
    │
    ▼
Generate test MAC
    │
    ▼
Build DHCP DISCOVER
    │
    ▼
Send packet
    │
    ▼
Repeat
```

---

## 🧵 Threading

The packet sniffer runs in a separate daemon thread:

```text
sniff_thread = threading.Thread(
    target=packet_sniffer,
    args=(args.iface, sniff_random_macaddr)
)
```
This allows the program to listen for DHCP responses while the main thread generates test traffic.

---

## 🌐 DHCP Packet Flow

A normal DHCP exchange generally follows:

```text
Client                         DHCP Server
  │                                 │
  │──── DHCP DISCOVER ─────────────►│
  │                                 │
  │◄──── DHCP OFFER ────────────────│
  │                                 │
  │──── DHCP REQUEST ──────────────►│
  │                                 │
  │◄──── DHCP ACK ──────────────────│
  │                                 │
```
This project focuses on demonstrating the packet-generation and response-observation portions of that process.

---

## 🛡️ Defensive Considerations

Understanding DHCP starvation is useful for network defenders because the same behavior can be detected and mitigated.

### DHCP Snooping

Managed switches can use DHCP snooping to distinguish legitimate DHCP-server traffic from unauthorized DHCP traffic.

### Port Security

Network administrators can restrict the number of MAC addresses allowed on an access port.

### Monitoring

Useful indicators can include:
- Large numbers of DHCP requests
- Rapidly changing client MAC addresses
- Unusual DHCP traffic bursts
- Unexpected DHCP activity on access ports
- Rapid depletion of the DHCP address pool

### Network Segmentation

Separating sensitive networks and controlling access to DHCP infrastructure can reduce the potential impact of a DHCP-based denial-of-service event.

---

## 🧪 Recommended Lab Topology

A simple isolated environment could look like:

```text
                    ISOLATED LAB
        ┌─────────────────────────────────┐
        │                                 │
        │       ┌─────────────────┐       │
        │       │  DHCP Server    │       │
        │       │  Test Network   │       │
        │       └────────┬────────┘       │
        │                │                │
        │                │                │
        │       ┌────────┴────────┐       │
        │       │  Test Machine   │       │
        │       │  Scapy Script   │       │
        │       └─────────────────┘       │
        │                                 │
        └─────────────────────────────────┘
```

Keep the environment disconnected from production networks.

---

## ⚠️ Limitations

This project is a simple educational demonstration, not a production network-testing framework.

Some limitations include:

- DHCP behavior can vary between server implementations.
- Packet structure assumptions may not work with every DHCP implementation.
- Network interfaces and operating systems can handle raw packets differently.
- The script has limited error handling.
- The script does not provide comprehensive lease-management functionality.
- It does not implement complete DHCP protocol validation.
- Network hardware may filter or modify certain packets.

---

## 🐛 Troubleshooting
Scapy import error

Install or update Scapy:

```text
python3 -m pip install --upgrade scapy
```

### Permission error

Packet capture and raw packet transmission generally require appropriate operating-system privileges.

### No DHCP traffic detected

Check that:

- The selected interface is correct.
- The interface is connected to the isolated test network.
- The DHCP server is running.
- DHCP traffic is reaching the interface.
- Your test environment permits the required broadcast traffic.

### DHCP server does not respond

Verify the DHCP server configuration and confirm that the server's address pool is available for the test client.

---

## 📚 Learning Objectives

After studying this project, you should have a better understanding of:

- DHCP discovery
- DHCP offers
- DHCP requests
- BOOTP headers
- DHCP transaction IDs
- Ethernet broadcast traffic
- MAC addresses
- Scapy packet construction
- Packet sniffing
- Network-layer troubleshooting
- DHCP starvation detection and mitigation

---

## 🔐 Responsible Use

This project is intended strictly for:

```text
✅ Authorized penetration testing
✅ Personal security laboratories
✅ Educational environments
✅ Network-security research
✅ Defensive testing
```

It should not be used for:

```text
❌ Attacking public networks
❌ Disrupting production infrastructure
❌ Exhausting another organization's DHCP pool
❌ Testing networks without authorization
```

Only run the software where you have explicit permission to generate network traffic.
