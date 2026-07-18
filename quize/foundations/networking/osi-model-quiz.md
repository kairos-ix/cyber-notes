# OSI Model — Quiz

Questions I got asked on this topic, written down so I can re-test myself later instead of just re-reading the note and assuming I remember it.

## How to use this

Don't scroll straight down. Read a question, answer it out loud like I'm in an interview, then open the answer and check. If I got it half right, that still counts as wrong — the point of this file is catching the gap between "I know the shape of it" and "I can say it precisely."

---

## Round 1 — Scenarios

**1. A user says "the internet is down." Their machine has a valid IP, can ping the gateway, but can't resolve google.com. Which layer do I suspect first, and which layers are already confirmed working?**

<details>
<summary>Answer</summary>

First suspect: **Application layer (Layer 7)** — DNS is an application-layer protocol, not presentation. Presentation is about data format (encryption, compression, encoding), not name resolution.

Confirmed working: **Layer 1, 2, and 3** — a successful ping to the gateway needs all three (raw signal, framing/MAC delivery, and IP routing to work). Don't credit a layer as "working" unless something in the scenario actually exercised it.

</details>

**2. Why does a successful ping prove Layer 1 through 3 are fine, not just Layer 3?**

<details>
<summary>Answer</summary>

Layer 3 picks the logical path via IP. Layer 2 wraps that into a frame and adds source/destination MAC so it can move on the local segment (ARP handles the IP→MAC lookup, error checking via CRC/FCS happens here too). Layer 1 turns that frame into raw bits and physically pushes them onto the wire/air — it has no concept of MAC addresses or frames, it just transmits blindly. Addressing and framing = Layer 2. Physical transmission = Layer 1. Don't merge them.

</details>

**3. Which OSI layer does ARP technically operate at?**

<details>
<summary>Answer</summary>

Genuinely doesn't map cleanly. Textbook answer is Layer 2, since its job (IP→MAC resolution) serves frame delivery. But ARP packets are encapsulated directly in Ethernet frames (EtherType 0x0806) — not carried inside IP — so it doesn't cleanly sit inside a single layer either. Often called "Layer 2.5." The senior answer isn't a clean label, it's knowing OSI doesn't fit every real protocol.

</details>

**4. Give a real scenario where I'd choose UDP over TCP, and say exactly what TCP feature I'm sacrificing to get that benefit.**

<details>
<summary>Answer</summary>

Video calls, live multiplayer gaming. A lost packet representing one video frame or one position update is worthless by the time TCP would retransmit it — the moment's already passed. UDP skips the handshake and retransmission, so we trade **reliability** for **low latency**. Not the other way around — we're not "trading speed to get dropped data," we're accepting dropped data to keep speed.

</details>

**5. What does TCP guarantee that UDP doesn't, in three words or fewer?**

<details>
<summary>Answer</summary>

**Reliability and ordering.** Not "accuracy" (that's just a restatement of reliability — both TCP and UDP use checksums for basic error detection). Definitely not "security" — TCP has zero built-in security. Encryption/auth comes from something layered on top, like TLS. Saying "TCP guarantees security" is a real red flag in an interview.

</details>

**6. On a video call, the connection drops for a few seconds then reconnects — same meeting, same participants, no re-auth. Which layer, and what's that layer's job in general?**

<details>
<summary>Answer</summary>

**Layer 5 — Session.** Job: establishing, maintaining, synchronizing, and terminating a session between two endpoints, independent of the actual data. The reason the call resumes instead of restarting is session-layer checkpointing/synchronization — it doesn't have to rebuild the whole session from zero.

</details>

**7. Is TLS actually a Layer 6 protocol, or is that a textbook simplification? Defend the answer.**

<details>
<summary>Answer</summary>

Textbook argument for Layer 6: presentation = formatting/encryption, and TLS encrypts data, so on paper it fits.

Real-world argument against: TLS runs directly on top of TCP, and it does more than formatting — handshake, certificate auth, session key negotiation, which lean more toward Layer 5/7 behavior. The TCP/IP model (what's actually used day to day) doesn't even have a Layer 5/6 — TLS is just "Application." Same pattern as ARP: OSI is a teaching model, real protocols straddle boundaries.

</details>

**8. Name three Layer 7 protocols and what each is for, five words or less.**

<details>
<summary>Answer</summary>

HTTP — transferring web content. SMTP — sending email between servers. FTP — transferring files.

Note: HTTPS isn't a separate protocol from HTTP — it's HTTP running over a TLS-encrypted connection. Don't list both as if they're unrelated.

</details>

**9. A switch operates primarily at which layer, and why not one layer lower?**

<details>
<summary>Answer</summary>

**Layer 2.** A switch forwards traffic based on MAC addresses — that requires frame-level info that only exists at Layer 2. A pure Layer 1 device (hub/repeater) has no addressing concept at all, it just repeats electrical signals to every port with zero intelligence about where data should go.

</details>

**10. A router operates at Layer 3 — what specific info does it read to forward traffic, and how is that different from a switch?**

<details>
<summary>Answer</summary>

A router reads the **destination IP address**, checks it against its **routing table**, and forwards out the right interface toward the next hop — potentially across different networks. A switch operates within a single local network and forwards based on MAC addresses, not IP. Core distinction: switches work **within** a network (Layer 2), routers connect **different** networks (Layer 3).

</details>

---

## Round 2 — Rapid fire

**1. What's the PDU called at Layer 4, and at Layer 3?**

<details>
<summary>Answer</summary>

Layer 4: **Segment** (TCP) or **Datagram** (UDP). Layer 3: **Packet**. (Layer 2 = Frame, Layer 1 = Bits.) Useful because saying "the packet got dropped" already tells a sharp engineer it's a Layer 3 issue without saying the layer number.

</details>

**2. A firewall doing basic port-based filtering (allow 443, block 8080) — which layer?**

<details>
<summary>Answer</summary>

**Layer 4** — ports are a transport-layer concept. Filtering by IP address alone would be Layer 3. Modern next-gen firewalls doing deep packet inspection on actual content push into Layer 7. The right answer depends on what it's actually filtering on.

</details>

**3. I unplug an Ethernet cable. Which single layer failed?**

<details>
<summary>Answer</summary>

**Layer 1.** The physical medium is gone.

</details>

**4. What does encapsulation mean, one sentence, Layer 7 down to Layer 1?**

<details>
<summary>Answer</summary>

Each layer adds its own header (sometimes a trailer) to the data from the layer above, without touching the original data — so by Layer 1 the original data is wrapped in nested headers, like nested envelopes. Ties directly to the PDU names: data → Segment → Packet → Frame → Bits.

</details>

**5. Two devices on the same LAN, correct IPs, cable's fine, still can't talk — ARP problem. Which layer, ultimately?**

<details>
<summary>Answer</summary>

**Layer 2.** ARP resolves IP→MAC, and MAC-based delivery on the local network is a Layer 2 job. Broken ARP resolution breaks Layer 2 delivery even though L1 (cable) and L3 (IP config) are both fine.

</details>

**6. What's the actual difference between a port number and an IP address — which layer, which problem does each solve?**

<details>
<summary>Answer</summary>

IP address (**Layer 3**) identifies which **device** on the network. Port number (**Layer 4**) identifies which **application/service on that device** the data is meant for. IP address = house address. Port = which door/apartment inside that house. Port has nothing to do with "the medium" — the medium is a Layer 1 concept.

</details>

**7. Why does TCP need a three-way handshake but UDP doesn't?**

<details>
<summary>Answer</summary>

The handshake (SYN, SYN-ACK, ACK) happens before any data is sent — it confirms both sides are reachable and **synchronizes sequence numbers**, which is what makes ordering and later retransmission possible at all. This is separate from retransmission itself, which happens during transfer. UDP skips it entirely because it doesn't track sequence numbers or connection state — it just sends and doesn't care if the other side is ready.

</details>

**8. A file arrives corrupted but with the correct size. Which layer catches it, and how?**

<details>
<summary>Answer</summary>

**Layer 4**, via TCP's checksum in the header. Receiver recalculates the checksum on arrival — if it doesn't match, the segment is discarded and TCP's retransmission kicks in (no ACK sent → sender times out → resends). Layer 2 also does error-checking via CRC/FCS in the frame trailer, but that's per-hop, not end-to-end like TCP's checksum.

</details>

**9. Bigger departure from classic Layer 4 behavior: HTTP/2 or QUIC (HTTP/3)?**

<details>
<summary>Answer</summary>

**QUIC.** HTTP/2 still runs on standard TCP — it changes Layer 7 behavior (multiplexing, header compression) but Layer 4 underneath is unchanged. QUIC runs on **UDP** and reimplements TCP-like reliability, ordering, and congestion control itself, at the application layer — blurring L4 and L7 on purpose. Faster in practice too: fewer round trips to set up (folds TLS negotiation in), and no head-of-line blocking, since one lost packet only stalls its own stream, not everything behind it.

</details>

**10. Someone says "OSI doesn't reflect how the real internet works, that's the TCP/IP model." Right or wrong, and why does it matter for interviews?**

<details>
<summary>Answer</summary>

Mostly right. OSI (7 layers) is a theoretical/reference model, never fully implemented as-is. TCP/IP (4 layers: Network Access, Internet, Transport, Application) is what the internet actually runs on — it squashes OSI's Session, Presentation, and Application into one Application layer. That's exactly why TLS, ARP, and QUIC don't map cleanly onto OSI — they were built around TCP/IP's simpler structure. Matters for interviews because when someone asks "which layer does X operate at" for something like TLS, they're often testing whether I understand OSI is a simplified model and can reason about where something approximately fits — not whether I can recite one "correct" box.

</details>
