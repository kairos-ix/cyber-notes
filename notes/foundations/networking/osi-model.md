# OSI Model
*Open Systems Interconnection*

## In short
A reference model that breaks down how data moves across a network into 7 layers, from the physical cable all the way up to the actual application.

## What it is
7 layers total. Numbering starts from the bottom — physical layer is layer 1, application layer is layer 7, not the other way around.

![OSI Model](../../../img/foundations/networking/osi-model.png)

## Quick reference

| # | Layer | Handles | PDU |
|---|---|---|---|
| 7 | Application | Interface the user actually interacts with — HTTP, DNS, SMTP | Data |
| 6 | Presentation | Format translation, compression, encryption | Data |
| 5 | Session | Starting, maintaining, ending sessions | Data |
| 4 | Transport | TCP/UDP, port numbers — hands data down to the network layer | Segment / Datagram |
| 3 | Network | IP addressing, routing across networks | Packet |
| 2 | Data Link | MAC addressing, framing, CRC error-checking | Frame |
| 1 | Physical | Cables, signals, connectors — raw transmission | Bits |

## Why it matters
This is the vocabulary I actually use when troubleshooting — "which layer is this problem at" is the real question behind most network issues.

→ Can't resolve a domain but the gateway pings fine? That's Layer 7 (DNS), not Layer 3 — the IP path already works, the app-layer lookup is what's failing.

→ Cable's unplugged? Layer 1, nothing higher even gets a chance to run.

→ Same LAN, correct IPs, still can't talk? Probably a Layer 2 problem — ARP not resolving MAC addresses correctly.

→ File arrives corrupted but the right size? Layer 4 territory — TCP's checksum catches this on arrival.

This also matters because it's the exact thing tested in a networking interview — not "recite the 7 layers," but "given this symptom, which layer, and why."

## How it works

### Encapsulation — the general flow
Before going layer by layer: as data moves down the stack, each layer adds its own header (sometimes a trailer) to whatever it received from the layer above, without touching the original data. By the time it hits Layer 1, the original application data is wrapped in nested headers — like nested envelopes. Going back up, each layer strips its own header off before passing the data up.

That's also where the PDU names come from: application data → **Segment** (Layer 4) → **Packet** (Layer 3) → **Frame** (Layer 2) → **Bits** (Layer 1).

---

### Layer 1 — Physical
This is the layer where the actual physical transmission happens — data moves through cables as **electrical signals**, or as **light** in fiber optic cables. It also covers the physical stuff itself: cable connectors, network cards, and so on.

→ Physical layer doesn't know what a MAC address is. It just pushes raw bits — no addressing, no framing, no intelligence about where anything should go. A hub is a pure Layer 1 device for this reason: it just repeats signals to every port.

---

### Layer 2 — Data Link
Mainly responsible for reliable data transfer between devices on the same local network, **MAC addressing**, and controlling who gets to send data and when.

→ It takes data from the network layer and **frames** it — the frame has the data itself, a header, and an error-checking value (**CRC**) in the trailer that helps detect if a frame got corrupted along the way.

→ **MAC (Media Access Control) address** is the address of the physical device itself — like a network card — used to identify the correct device on the local network.

→ A **switch** is a Layer 2 device — it forwards frames based on MAC address, not IP. That's a different job from a router (see Layer 3).

→ **ARP (Address Resolution Protocol)** is what resolves an IP address to a MAC address — broadcasts "who has this IP" on the network, the owning device replies with its MAC. It doesn't cleanly fit one OSI layer though: ARP packets are carried directly inside Ethernet frames, not inside IP packets, so it's often called "Layer 2.5." Worth knowing it's a case where a real protocol doesn't respect the OSI boundary.

---

### Layer 3 — Network
Mainly handles **IP addressing**, routing data between different networks through routers, and subnet masking.

→ It takes data from the transport layer and wraps it into **packets**, adding source and destination IP addresses, so it can be routed to a different network — not just the local one, that part is the data link layer's job.

→ A **router** is a Layer 3 device — it reads the destination IP, checks it against its routing table, and forwards out the right interface toward the next hop. That's the core difference from a switch: switches operate *within* one local network (Layer 2, MAC-based), routers connect *different* networks *together* (Layer 3, IP-based).

---

### Layer 4 — Transport
Sits between the session layer above it and the network layer below it — hands data down toward the network layer, and reassembles it on the way back up.

→ Uses **TCP** (Transmission Control Protocol) and **UDP** (User Datagram Protocol) for transmission of data.

→ **Port numbers** live here, not at Layer 3. IP address identifies *which device* on the network — port number identifies *which application/service on that device* the data is meant for. IP address = the house address, port = which door inside the house.

→ TCP starts with a **three-way handshake** (SYN, SYN-ACK, ACK) before any data moves. Its real job isn't just "saying hello" — it **synchronizes sequence numbers** between both sides, which is what makes ordering and retransmission possible later. UDP skips this entirely — no handshake, no sequence tracking, it just sends.

→ What TCP actually guarantees: **reliability and ordering** — that's it. Not "security" (that's TLS, layered on top, not a TCP feature). If a segment gets corrupted, TCP's own **checksum** in the header catches it on arrival — receiver recalculates and compares, mismatch means the segment gets discarded and retransmission kicks in.

→ UDP trades that reliability away *for* speed — no handshake, no retransmission, no ordering guarantee. Used where a late packet is worse than a lost one: video calls, live gaming.

---

### Layer 5 — Session
Responsible for managing sessions — starting, maintaining, and ending a session between two devices.

→ Before any data transfer happens, a session needs to be established first.

→ Also handles synchronization/checkpointing — this is why a video call can pick back up mid-meeting after a brief network drop instead of restarting the whole session from scratch.

---

### Layer 6 — Presentation
Mainly responsible for translating data into a format the application layer can actually understand. Format conversion and **compression** happen here too.

→ **Encryption** also happens at this layer, on paper — but TLS is the common real-world example, and it's usually classified as an **application-layer** protocol in practice, not Layer 6. It runs on top of TCP and does more than just formatting — handshake, certificate authentication, session key negotiation. The TCP/IP model (what real networking actually runs on) doesn't even have a separate Layer 5/6 — it's all just "Application." Same pattern as ARP: OSI-textbook answer vs. real-world answer aren't always the same thing.

---

### Layer 7 — Application
This is the interface layer the user actually interacts with. Includes protocols like **HTTP**, **DNS**, **SMTP**, **FTP**.

→ **HTTPS isn't a separate protocol** from HTTP — it's HTTP running over a TLS-encrypted connection.

→ **DNS** belongs here too — resolving a domain name to an IP is an application-layer job, not presentation. Easy to mix up because "translation" sounds like a presentation-layer word, but DNS is a protocol an application uses, not a data-format conversion.

---

## OSI vs. the real internet
The internet doesn't actually run on OSI. It runs on the **TCP/IP model**, which only has 4 layers: Network Access (combines Physical + Data Link), Internet (= Network), Transport, and Application (combines Session + Presentation + Application).

OSI was built as a teaching/reference model, largely after TCP/IP already existed and won. That's the actual reason ARP, TLS, and newer protocols like **QUIC** (used in HTTP/3, runs on UDP but reimplements TCP-like reliability and ordering at the application layer) don't map cleanly onto OSI's 7 boxes — they were designed around TCP/IP's simpler structure, not OSI's. When something doesn't fit one clean layer, that's not a gap in my understanding — that's just OSI being a simplified model.

## Key details to remember
- 7 layers total, numbered bottom to top: Physical(1) → Data Link(2) → Network(3) → Transport(4) → Session(5) → Presentation(6) → Application(7)
- PDU names bottom to top: Bits(L1) → Frame(L2) → Packet(L3) → Segment/Datagram(L4)
- Data Link = frames, MAC address + CRC. Device: switch.
- Network = packets, IP address + routing table. Device: router.
- Transport = TCP (reliable, ordered, handshake) / UDP (fast, no guarantee). Ports live here.
- TCP guarantees = reliability + ordering only. Not security — that's TLS.
- ARP = IP→MAC resolution. Ethernet-framed, not IP-carried — doesn't cleanly fit one layer ("Layer 2.5").
- TLS = textbook Layer 6, real-world usually just "Application."
- HTTPS = HTTP + TLS, not its own protocol.
- Real internet runs on the 4-layer TCP/IP model. OSI is a teaching model, not the actual architecture.
- Full name: Open Systems Interconnection

## Where I got confused
- Thought DNS was a presentation-layer thing since "resolving a name" sounded like translation. It's application layer — DNS is a protocol an app uses, not a format conversion.
- Mixed up Layer 1 and 2 — said frames get sent to a MAC address "via the physical layer." Physical layer doesn't know what a MAC address is, it just pushes bits. Addressing and framing are Layer 2's job entirely.
- Couldn't commit to a layer for ARP, kept describing what it does instead of where it sits. Turns out that's fine — it genuinely doesn't sit in one clean box, since it's Ethernet-framed, not IP-carried.
- Explained the UDP-over-TCP tradeoff backwards at first — said we trade "speed for dropped data" when it's the other way around: we give up reliability to protect speed.
- Listed TCP's guarantees as "reliability, accuracy, security." Accuracy was just reliability restated, and security isn't TCP's job at all — that's TLS, layered on top.
- Called TLS "application layer, for security" without being able to defend why. Right layer in practice, but no reasoning behind it — reasoning is the part that actually matters in an interview.
- Talked about HTTP and HTTPS like they were two separate Layer 7 protocols. HTTPS is just HTTP over TLS.
- Described a router as reading "the next hop address" — that's the output of its decision, not what it reads. It reads the destination IP and checks the routing table.
- Called a port number "the medium" at first. Medium is a Layer 1 idea — port identifies the service on the device, has nothing to do with how the signal physically travels.

## How I'd say this out loud
OSI is a 7-layer model for how data gets from one device to another — physical wires at the bottom, the actual app at the top. Going down the stack, each layer wraps the data from the layer above in its own header, that's encapsulation, so a chunk of app data becomes a segment, then a packet, then a frame, then just bits on the wire. Devices map to layers too — a hub is Layer 1, a switch is Layer 2 because it forwards on MAC address, a router is Layer 3 because it forwards on IP using a routing table. The catch is the real internet doesn't actually run on OSI — it runs on the simpler 4-layer TCP/IP model, and OSI is really just a teaching tool. That's why protocols like ARP or TLS never sit cleanly in one OSI box — they were built for TCP/IP, not for OSI.