# 🛡️ SOC Analyst Learning Journey

Welcome to my daily documentation tracker as I build hands-on skills for a Security Operations Center (SOC) Analyst role. 

---

## 📅 Day 1: Network Security Foundations

### 🧠 Core Concept: What is a Network & Data Movement
A network is fundamentally just computers talking to each other. It is not an abstract "cloud"—it consists of physical assets: your device, a local router, an Internet Service Provider (ISP), and destination servers.

* **The Packet:** Data does not travel all at once; it is broken down into small pieces called **packets** (like an envelope).
* **Packet Structure:** Contains a **Source IP** (where it comes from), a **Destination IP** (where it is going), and the **Payload** (the actual data/request).
* **The Security Perspective:** *Every network attack is just modified, suspicious, fake, or stolen packets.* Understanding normal packet flow is the prerequisite to identifying malicious packet behavior.

---

### 📦 The Real Path of a Packet (Routing)
When requesting data from a server (e.g., watching a video on YouTube), data moves through a multi-step delivery system:

1.  **Device (Start Point):** Generates the request and packages it into a packet.
2.  **Router (Wi-Fi Box):** Acts as the local traffic manager, deciding where to send the packet next.
3.  **ISP (Internet Service Provider):** Connects the local home network to the global internet infrastructure.
4.  **Internet Routers (Hops):** The packet moves from router to router across the globe via a process called **Routing**.
5.  **Destination Server:** The website receives the packet, processes the request, and sends response packets back along the reverse path.

> ⚠️ **Security Engineering Insight:** Understanding this path allows analysts to detect the origin of attacks, identify where traffic is intercepted, and spot Man-In-The-Middle (MITM) or packet-sniffing exploits.

---

### 🏗️ The OSI Model: Mapping the Network
The OSI Model is a 7-layer map that defines where specific network actions—and security threats—occur. 

| Layer | Name | Core Function | Security Context |
| :--- | :--- | :--- | :--- |
| **Layer 7** | Application | What the user interacts with (Browsers, Apps) | Web exploits, login forms, SQL Injection, XSS |
| **Layer 4** | Transport | Controls delivery rules (**TCP** / **UDP**) | Port scanning, managing open communication ports |
| **Layer 3** | Network | Handles routing and **IP addresses** | IP spoofing, routing attacks |
| **Layer 2** | Data Link | Local network communication (**MAC addresses**) | Local sniffing, ARP poisoning attacks |
| **Layer 1** | Physical | Physical signals (Wi-Fi waves, Ethernet cables) | Physical breaches, hardware tampering |

### 🛠️ Day 1 Mental Walkthrough
When connecting to a secure website via Wi-Fi:
* **Browser (Chrome/Firefox)** operates at **Layer 7**.
* **HTTPS (TCP connection)** rules operate at **Layer 4**.
* **IP Routing** across the internet operates at **Layer 3**.
* **Wi-Fi connection** to the home router operates at **Layer 2**.
* **Radio waves** transmitting the data operate at **Layer 1**.

* Add Day 1 Notes



## 📅 Day 2: Network Visibility & Protocols (Wireshark, DNS, & TCP)

### 🔬 Core Tool: Wireshark (Network Traffic Analysis)
Moving from theory to reality requires packet visibility. **Wireshark** acts as a network packet analyzer (effectively a "CCTV camera" for network traffic), capturing and displaying data packets traveling across a network interface in real time.

* **Network Interfaces:** The point where a device connects to a network (e.g., `Wi-Fi` for wireless, `Ethernet` for wired cables, or `Loopback` for internal system testing).
* **The Baseline Noise:** Live networks are constantly generating "noisy" background traffic. Analyzing traffic requires filtering out the noise to focus on specific protocols, source IPs, or anomalous indicators.
* **The Security Perspective:** Wireshark is the fundamental tool for traffic monitoring, intrusion detection, and incident response. Without packet visibility, a SOC analyst is blind to active threats.

---

### 📖 Protocol Focus: DNS (The Internet Phonebook)
Computers do not communicate via human-readable domain names (like `google.com`); they communicate via numerical IP addresses. The **Domain Name System (DNS)** resolves domain names into routable IP addresses.



* **Packet Filtering:** In Wireshark, applying the display filter `dns` isolates domain queries and responses.
* **DNS Record Structure:** A standard query seeks an **A Record** (which maps a hostname to an IPv4 address).
* > ⚠️ **Security Engineering Insight:** DNS traffic often remains unencrypted even if the subsequent web traffic is encrypted via HTTPS/TLS. Attackers frequently target DNS through **DNS Poisoning** or malicious redirection. For an analyst, DNS logs reveal exactly what external domains an internal asset is attempting to contact *before* an encrypted session ever begins.

---

### 🤝 Protocol Focus: The TCP 3-Way Handshake
Before data can reliably move between a client and a server, the **Transmission Control Protocol (TCP)** must establish a connection. This connection is opened using a mandatory, structured process called the **3-Way Handshake**.
Client                               Server
|                                    |
| ------------ SYN ------------->    |  (Can we connect?)
|                                    |
| <---------- SYN-ACK -----------    |  (Yes, I am ready. Are you?)
|                                    |
| ------------ ACK ------------->    |  (Confirmed. Connection open.)
|                                    |


* **Step 1: SYN (Synchronize):** The client sends a packet with the SYN flag set to request a connection.
* **Step 2: SYN-ACK (Synchronize-Acknowledge):** The server responds, confirming receipt of the request and asking to synchronize back.
* **Step 3: ACK (Acknowledge):** The client sends a final confirmation packet. The connection is now established, and data (such as web pages) can safely flow.

#### 🧠 Analyst Thinking: Analyzing the Handshake in Wireshark
By applying the `tcp` filter in Wireshark, we can audit these handshakes to diagnose network health or malicious intent:
* **Normal Behavior:** A clean sequence of `SYN` ➔ `SYN-ACK` ➔ `ACK`.
* **Suspicious Behavior (Reconnaissance):** A flood of rapid `SYN` packets to multiple different ports without ever completing the handshake with an `ACK`. This is a classic **SYN Port Scan** used by attackers to map out open vulnerabilities on a target system.
* **Denial of Service (DoS):** Flooding a server with endless `SYN` requests to exhaust its resources keeping half-open connections waiting.

* Add Day 2 Notes


