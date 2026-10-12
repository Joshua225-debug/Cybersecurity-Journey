# Networking Fundamentals: My Note

> Written so that future-me can read this cold and still understand it.
> Covers theory (TryHackMe Pre Security + deep chat teaching) and my hands-on build in Cisco Packet Tracer.

**How I learn:** I predict the result BEFORE I run a command, run it, then compare. When I'm wrong, that's where I learn the most.

---

## 1. The Big Picture

A **network** is just two or more devices connected so they can share data.

Every time I load a website, a few things happen:

1. My computer asks **DNS**: "what is the IP address of this name?"
2. My computer builds a **packet** addressed to that IP.
3. The packet travels through my **switch**, my **router**, and many other routers.
4. The server replies, and the data comes back the same way.

Think of it like sending a letter. The **IP address** is the building. The **port** is the door. The **MAC address** is the person's name on the mailbox at the final step.

## 2. Addresses: IP and MAC

### IP address (Layer 3, logical)

- Identifies a device on a network. It can change.
- **IPv4** looks like `192.168.1.10` (four numbers, 0 to 255, separated by dots). It is 32 bits long.
- **IPv6** is longer, like `2a09:bac1:5400:30::3d8:50`. It was created because IPv4 addresses are running out.

### Private vs public IPs

| Type | Ranges | Where used |
|------|--------|-----------|
| Private | `10.0.0.0 to 10.255.255.255`, `172.16.0.0 to 172.31.255.255`, `192.168.0.0 to 192.168.255.255` | Inside home/office networks |
| Public | Everything else | The open internet |

My Packet Tracer networks use `192.168.x.x`, which is private.

### Special addresses

- `127.0.0.1` is **loopback**. It means "this very computer". Pinging it tests my own network software.
- `255.255.255.255` is **broadcast**: "everyone on this network".

### MAC address (Layer 2, physical)

- Burned into the network card. Looks like `AA:BB:CC:11:22:33`.
- 48 bits, written in hexadecimal.
- Only matters on the **local network**. Routers swap MAC addresses at every hop, but the IP stays the same from source to destination.

**Key idea:** IP = where you are going. MAC = who is next, one hop away.

## 3. Subnets and Masks

A **subnet** is a smaller network carved out of a bigger one.

A **subnet mask** tells a device which part of an IP is the *network* and which part is the *host*.

Example: 255.255.255.0 (also written /24)

IP:    192.168.1.10
Mask:  255.255.255.0
         network part = 192.168.1
                  host part    = .10

- /24 means the first 24 bits are the network. That leaves 8 bits for hosts.
- 8 host bits gives 256 addresses, but 2 are reserved (network address .0 and broadcast .255), so **254 usable hosts**.
- Two devices talk **directly** only if they are in the same subnet. Otherwise they must go through the **default gateway** (the router).

In my Packet Tracer build:

- LAN 1 is 192.168.1.0/24
- LAN 2 is 192.168.2.0/24

Two different subnets, which is exactly why they need a router between them.

## 4. Network Devices

| Device | Works with | Job |
|--------|-----------|-----|
| **Hub** | Layer 1 | Repeats data to every port. Dumb, old, noisy. |
| **Switch** | Layer 2 (MAC) | Connects devices in the **same** LAN. Learns which MAC is on which port and sends data only where needed. |
| **Router** | Layer 3 (IP) | Connects **different** networks. Reads IP addresses and decides the best path. |
| **Firewall** | Layers 3 to 7 | Allows or blocks traffic based on rules. |
| **Modem** | Layer 1 | Converts signals between my network and my ISP. |

**Easy way to remember:**
- Switch = connects things *inside* one network.
- Router = connects *different* networks together.

### Cables used in Packet Tracer

- **Straight-through**: connects *different* kinds of devices (PC to switch, router to switch).
- **Crossover**: connects *similar* kinds of devices (PC to PC, switch to switch).

## 5. The OSI Model

A 7-layer model that describes how data moves across a network. Top to bottom:

| # | Layer | What it does | Example |
|---|-------|--------------|---------|
| 7 | **Application** | What the user's software talks to | HTTP, DNS, FTP |
| 6 | **Presentation** | Formatting, encryption, compression | TLS, JPEG |
| 5 | **Session** | Opens, manages, closes conversations | Sessions, cookies |
| 4 | **Transport** | Splits data, adds ports, reliability | TCP, UDP |
| 3 | **Network** | Logical addressing, routing | IP, ICMP, routers |
| 2 | **Data Link** | Local delivery using MAC addresses | Ethernet, switches |
| 1 | **Physical** | Actual signals, cables, Wi-Fi | Cables, radio waves |

**Memory trick (top to bottom):** All People Seem To Need Data Processing

When data is **sent**, it goes down the layers and gets wrapped at each one. That is called **encapsulation**. When data is **received**, it goes up and is unwrapped. That is **de-encapsulation**.

## 6. The TCP/IP Model

The simpler, real-world model the internet actually uses. 4 layers:

| TCP/IP Layer | Matches OSI | Examples |
|--------------|-------------|----------|
| **Application** | 5, 6, 7 | HTTP, HTTPS, DNS, FTP, SSH |
| **Transport** | 4 | TCP, UDP |
| **Internet** | 3 | IP, ICMP, ARP |
| **Network Access** | 1, 2 | Ethernet, Wi-Fi |

**OSI vs TCP/IP:** OSI is a teaching model (7 layers). TCP/IP is the practical model (4 layers). They describe the same journey in two ways.

## 7. Packets and Frames

Data is not sent in one big lump. It is chopped into small pieces.

- A **packet** is a piece of data at **Layer 3**. It carries source IP and destination IP.
- A **frame** is that packet wrapped for **Layer 2**. It carries source MAC and destination MAC.

Order inside a frame: Frame header (MACs), then Packet header (IPs), then Segment header (ports), then the DATA, then the Frame trailer.

Why split data up? So that many conversations can share the same cable, and if one piece is lost only that piece is resent.

**What changes at a router:** the frame is thrown away and a new frame is built with new MAC addresses. The packet inside, with the same IPs, goes on.

## 8. TCP vs UDP

Both live at the **Transport layer**. Both use ports.

### TCP (Transmission Control Protocol)

- **Reliable.** Checks that everything arrived, in order.
- Starts with the **three-way handshake**: Client sends SYN ("Can we talk?"), Server replies SYN/ACK ("Yes, can you hear me?"), Client sends ACK ("Yes. Let's go.")
- Slower, but safe. Used for: web (HTTP/HTTPS), email, SSH, file transfer.

### UDP (User Datagram Protocol)

- **Fast, no guarantees.** No handshake, no checking, no resending.
- Used for: video calls, gaming, streaming, DNS lookups.

| | TCP | UDP |
|---|-----|-----|
| Reliable | Yes | No |
| Handshake | Yes | No |
| Speed | Slower | Faster |
| Good for | Web pages, files | Calls, games, streaming |

## 9. Ports

An IP address finds the **computer**. A **port** finds the **program** on that computer.

- Ports are numbers from **0 to 65535**.
- A **socket** is IP + port together, like `192.168.1.10:80`.

### Ports worth knowing

| Port | Protocol | Use |
|------|----------|-----|
| 20/21 | FTP | File transfer |
| 22 | SSH | Secure remote terminal |
| 23 | Telnet | Old, insecure remote terminal |
| 25 | SMTP | Sending email |
| 53 | DNS | Name lookups |
| 67/68 | DHCP | Getting an IP automatically |
| 80 | HTTP | Web (not encrypted) |
| 110 | POP3 | Receiving email |
| 143 | IMAP | Receiving email |
| 443 | HTTPS | Web (encrypted) |
| 3389 | RDP | Windows remote desktop |

### Port ranges

- **0 to 1023**: well-known ports (reserved for standard services)
- **1024 to 49151**: registered ports
- **49152 to 65535**: dynamic/ephemeral ports (my computer picks one at random when it opens a connection)

### Port states (matters later for Nmap)

- **Open**: a program is listening.
- **Closed**: reachable, but nothing is listening.
- **Filtered**: a firewall is hiding the answer.

**Security link:** every open port is a possible way in. Fewer open ports means a smaller attack surface.

## 10. ARP

**ARP (Address Resolution Protocol)** answers the question: "I know the IP address. What is the MAC address?"

How it works inside a LAN:

1. My PC wants to talk to `192.168.1.1` but doesn't know its MAC.
2. It **broadcasts**: "Who has 192.168.1.1? Tell me your MAC."
3. The owner replies with its MAC.
4. My PC stores the answer in its **ARP table (cache)** for next time.

Command to see the ARP table: arp -a

**Why the first ping often times out:** ARP has to run first. That takes a moment, so the very first request may be lost. The next ones work. I saw this in Packet Tracer.

**Security link:** ARP has no authentication, so attackers can lie to it. That is called **ARP spoofing**.

## 11. ICMP and Ping

**ICMP (Internet Control Message Protocol)** is the network's way of sending status and error messages. It sits at Layer 3.

**Ping** uses two ICMP messages:

- **Echo Request** (type 8): "Are you there?"
- **Echo Reply** (type 0): "Yes, I'm here."

Commands: ping 127.0.0.1 tests my own machine. ping 192.168.1.1 tests my gateway.

### TTL (Time To Live)

Every packet carries a TTL number. **Every router it passes subtracts 1.** At 0, the packet is dropped.

Why? So a packet stuck in a loop can't circle forever.

**My real example:** Windows starts at TTL 128. My ping from PC0 to PC2 crossed 1 router, so the reply showed **TTL=127**. That one missing number proved one hop.

Other ICMP uses: traceroute (finds each hop on the path) and "destination unreachable" errors.

## 12. Routing

**Routing** is deciding where to send a packet next so it reaches a different network.

### Default gateway

The router address a device uses for **anything outside its own subnet**.

- PC in `192.168.1.0/24` has gateway `192.168.1.1`.
- If a device has no gateway, it can talk to neighbours but **not** to the outside world.

### Routing table

A router keeps a list: "for this destination network, send out of this interface / to this next hop."

- **Directly connected** networks appear automatically once the interface is up and has an IP.
- **Static routes** are added by hand.
- **Dynamic routing** (like RIP, OSPF) lets routers share routes with each other. I have not built this yet.

### How a packet crosses a router

1. PC0 sees the destination is outside its subnet.
2. PC0 sends the frame to its **gateway's MAC**.
3. The router reads the destination IP, looks at its routing table, picks the exit interface.
4. The router builds a **new frame** for the next hop and subtracts 1 from TTL.
5. The destination replies, and the same happens in reverse.

## 13. DHCP

**DHCP (Dynamic Host Configuration Protocol)** hands out network settings automatically so nobody types IPs by hand.

It gives out: IP address, subnet mask, default gateway, DNS server.

### The DORA process

D - Discover: Client broadcasts "Is there a DHCP server?"
O - Offer: Server replies "Here, you can use this IP."
R - Request: Client says "Yes, I'll take that one."
A - Acknowledge: Server confirms. Lease starts.

- Uses UDP ports **67** (server) and **68** (client).
- The address is **leased** for a time, then renewed.
- **Static IP** = typed by hand (what I did in Packet Tracer so far). **DHCP** = automatic.

## 14. DNS

**DNS (Domain Name System)** is the internet's phonebook. It turns a name like `tesla.com` into an IP address.

### How a lookup works

1. My computer checks its own **cache**.
2. It asks its **recursive resolver** (usually my ISP or `8.8.8.8`).
3. The resolver asks a **root server**, then the **TLD server** (`.com`), then the **authoritative server** for the domain.
4. The answer comes back and is cached.

### Common record types

| Record | Meaning |
|--------|---------|
| **A** | Name to IPv4 address |
| **AAAA** | Name to IPv6 address |
| **CNAME** | Alias pointing to another name |
| **MX** | Mail servers for the domain |
| **NS** | The name servers for the domain |
| **TXT** | Free text (often used for verification and email security) |

- DNS uses port **53** (UDP mostly, TCP for big answers).
- Tools: nslookup, dig, whois.

**Security link:** DNS records reveal subdomains and mail servers. That is passive reconnaissance material.

## 15. HTTP and HTTPS

**HTTP (HyperText Transfer Protocol)** is how browsers and web servers talk. It is a request and response system.

### A request has

- **Method**: what I want to do
- **Path**: which resource
- **Headers**: extra info (browser type, cookies)
- Sometimes a **body** (like form data)

### Methods

| Method | Meaning |
|--------|---------|
| GET | Fetch something |
| POST | Send data |
| PUT | Update or replace |
| DELETE | Remove |

### Status codes

| Code | Meaning |
|------|---------|
| 200 | OK |
| 301/302 | Redirect |
| 400 | Bad request |
| 401 | Not authorised (need to log in) |
| 403 | Forbidden |
| 404 | Not found |
| 500 | Server error |

Rule of thumb: **2xx** success, **3xx** redirect, **4xx** my mistake, **5xx** server's mistake.

### HTTPS

HTTPS = HTTP + **TLS encryption**. Port **443**.

- Stops people on the network from reading or changing the data.
- The padlock in the browser means the connection is encrypted. It does **not** mean the website is trustworthy.

### Cookies and sessions

HTTP forgets me after each request. **Cookies** let a site remember me (for example, that I'm logged in).

## 16. Firewalls

A **firewall** allows or blocks traffic using **rules**.

- Rules look at source IP, destination IP, port, and protocol.
- **Allow list**: only listed things get through.
- **Deny list**: listed things are blocked.
- **Stateless** firewalls judge each packet alone. **Stateful** firewalls remember ongoing connections.
- A **host firewall** protects one machine. A **network firewall** protects a whole network.

Related ideas: **IDS** detects suspicious traffic and alerts. **IPS** detects and blocks.

## 17. Kali Practice: First Commands

My Kali VM runs in VirtualBox. These are read-only, safe commands.

ip a                 - show all network interfaces and their IPs
ping -c 4 127.0.0.1  - loopback test: talks to myself
ping -c 4 10.0.2.2   - my VirtualBox gateway
arp -a               - ARP table (or: ip neigh)

What I learned:

- `lo` is the **loopback** interface (`127.0.0.1`).
- `eth0` is my real network interface in the VM.
- `10.0.2.2` is the gateway VirtualBox gives a NAT network.
- `-c 4` means "send only 4 pings".

## 18. Cisco Packet Tracer: My Build

Plan: 6 stages. **Stages 1 to 3 are done.** The rest are in section 20, Still To Come.

### Stage 1: Interface and devices

Learned the Packet Tracer workspace: the device palette at the bottom, drag-and-drop, and cable types.

### Stage 2: LAN 1

PC0 (192.168.1.10) and PC1 (192.168.1.11) connected to Switch0.

- Cables: **straight-through** (PC to switch).
- Mask: `255.255.255.0`.
- On each PC: Desktop, IP Configuration, **Static**, type IP and mask.
- PC0 and PC1 can ping each other, because they are in the same subnet.

### Stage 3: Router and LAN 2

Added a **Cisco 1941** router (Router1) and a second LAN.

LAN 1 (192.168.1.0/24): PC0 at 192.168.1.10 and PC1 at 192.168.1.11, connected to Switch0.
Switch0 connects to Router1's Gig0/0 interface at 192.168.1.1.
Router1's Gig0/1 interface at 192.168.2.1 connects to Switch1.
LAN 2 (192.168.2.0/24): PC2 at 192.168.2.10 and Laptop0 at 192.168.2.11, connected to Switch1.

| Device | IP | Default gateway |
|--------|----|-----------------|
| PC0 | 192.168.1.10 | 192.168.1.1 |
| PC1 | 192.168.1.11 | 192.168.1.1 |
| Router Gig0/0 | 192.168.1.1 | n/a |
| Router Gig0/1 | 192.168.2.1 | n/a |
| PC2 | 192.168.2.10 | 192.168.2.1 |
| Laptop0 | 192.168.2.11 | 192.168.2.1 |

All masks are `255.255.255.0`.

### Router CLI: step by step

Cisco CLI has **modes**. The prompt shows which one I'm in:

| Prompt | Mode |
|--------|------|
| `Router>` | User EXEC (very limited) |
| `Router#` | Privileged EXEC (can see everything) |
| `Router(config)#` | Global configuration |
| `Router(config-if)#` | Interface configuration |

Commands I used:

Router> enable  -  go to privileged mode
Router# configure terminal  -  go to config mode
Router(config)# interface gigabitEthernet 0/0  -  pick the interface
Router(config-if)# ip address 192.168.1.1 255.255.255.0
Router(config-if)# no shutdown  -  TURN THE PORT ON
Router(config-if)# exit

Router(config)# interface gigabitEthernet 0/1
Router(config-if)# ip address 192.168.2.1 255.255.255.0
Router(config-if)# no shutdown
Router(config-if)# end

Router# show ip interface brief  -  verify

**What show ip interface brief should show:** both interfaces with their IPs and Status up, Protocol up.

### The big test

Ping from PC0 to PC2 (different networks): ping 192.168.2.10

Result: **3 of 4 replies, TTL=127.**

- **Why not 4 of 4?** The first request timed out because ARP had to find the MAC address first.
- **Why TTL=127?** Windows starts at 128 and the router subtracted 1.

That is my packet crossing a router for the first time.

## 19. Mistakes That Taught Me Things

1. **Red triangles everywhere.** Router interfaces are **off by default** ("administratively down"). Fix: no shutdown. Lesson: a red light is a clue, not a failure.

2. **Missing default gateway.** Without one, a device can reach its neighbours but nothing outside its subnet. Always check the gateway first when cross-network pings fail.

3. **First ping timing out.** Not a bug. ARP was doing its job.

4. **Wrong cable type.** Straight-through vs crossover matters. Same-type devices need crossover.

**Troubleshooting order I'm building as a habit:**

1. Is the cable connected and the light green?
2. Is the IP and mask correct?
3. Is the default gateway set?
4. Is the router interface up? (show ip interface brief)
5. Can I ping my own gateway? Then the other side?

## 20. Still To Come

Not done yet in Packet Tracer:

- **Stage 4: DHCP**: set up the router or a server to hand out addresses automatically.
- **Stage 5: Servers and services**: DNS server and web server.
- **Stage 6: CLI configuration and troubleshooting**: break things on purpose and fix them.

I'll add each stage here when I actually do it.

## 21. Glossary

| Term | Plain meaning |
|------|---------------|
| **ARP** | Finds a MAC address from an IP |
| **Broadcast** | A message to everyone on the network |
| **CLI** | Command-line interface |
| **DHCP** | Gives out IP settings automatically |
| **DNS** | Turns names into IPs |
| **Encapsulation** | Wrapping data in headers as it goes down the layers |
| **Frame** | Layer 2 unit of data |
| **Gateway** | The router a device uses to leave its network |
| **HTTP/HTTPS** | Web protocol / encrypted web protocol |
| **ICMP** | Messages for status and errors (ping uses it) |
| **LAN** | Local area network |
| **Loopback** | 127.0.0.1, meaning "me" |
| **MAC** | Hardware address on the network card |
| **NAT** | Translating private IPs to a public IP |
| **Packet** | Layer 3 unit of data |
| **Port** | A number that identifies a program |
| **Router** | Connects different networks |
| **Subnet** | A smaller section of a network |
| **Switch** | Connects devices in one LAN using MACs |
| **TCP** | Reliable transport |
| **TTL** | Hop limit on a packet |
| **UDP** | Fast, unreliable transport |

---

*Source material: TryHackMe Pre Security (What is Networking, Intro to LAN, OSI Model, Packets and Frames, Extending Your Network), Cisco Packet Tracer practice, and Kali terminal exercises.*
