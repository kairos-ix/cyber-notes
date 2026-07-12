# Networking Devices — Quiz

Questions I got asked on this topic, written down so I can re-test myself later instead of just re-reading the note and assuming I remember it.

## How to use this

Don't scroll straight down. Read a question, answer it out loud like I'm in an interview, then open the answer and check. If I got it half right, that still counts as wrong — the point of this file is catching the gap between "I know the shape of it" and "I can say it precisely."

---

## Round 1 — Scenarios

**1. A user can reach every device on their own subnet without issue, but nothing outside it. Which device is actually responsible, what layer does it operate at, and why does that layer matter for this specific symptom?**

<details>
<summary>Answer</summary>

The **router**. Communication within the same subnet only needs Layer 2 (MAC) forwarding — devices ARP for each other and the switch forwards locally, no router involved. Crossing into another subnet needs Layer 3 (IP) forwarding: the source device notices the destination IP is outside its own subnet mask and sends the packet to its **default gateway** — a router — which decides where to send it next via its routing table. So "same subnet fine, different subnet fails" almost always means a missing/misconfigured default gateway or a routing table issue, not a switch problem. A broken switch would usually kill local connectivity too, not selectively break only cross-subnet traffic.

</details>

**2. Three VLANs live on the same physical switch, and two of them need to talk to each other. What device is required, what's this setup called, and why can't the switch handle it alone even though everything's on the same hardware?**

<details>
<summary>Answer</summary>

A **router** (or a Layer 3/multilayer switch) is required. When done with a router and a trunk port, the setup is called **router-on-a-stick** — inter-VLAN routing. VLANs are, by design, separate broadcast domains/subnets even when they share the same physical switch. A switch operating purely at Layer 2 forwards by MAC address and has no concept of IP addressing or routing, so it treats each VLAN as an entirely separate network it can't bridge on its own. Router-on-a-stick uses a single trunk port carrying tagged traffic for multiple VLANs (802.1Q), with the router holding a sub-interface per VLAN to actually route between their subnets.

</details>

**3. Multiple access points need central management, firmware pushes, and policy changes from one place, plus visibility into which ones are underperforming. Name the device, and explain the difference between the deployment model it enables and configuring each AP individually.**

<details>
<summary>Answer</summary>

A **WLAN controller**. Configuring each AP by hand is the **fat AP** (autonomous AP) model — each device holds its own full configuration and control-plane intelligence. A WLAN controller enables the **thin AP** model instead — APs hold minimal configuration and lean on the controller for control-plane decisions (channel selection, transmit power, roaming coordination), mostly just handling the radio layer themselves. Beyond saving admin time, the controller also does **RF coordination** between neighboring APs — adjusting channels/power to cut co-channel interference — and coordinates **fast roaming** so clients don't drop or fully re-authenticate moving between APs. That's the real technical reason centralization matters at scale, not just convenience.

</details>

**4. Two offices need encrypted traffic between them over the public internet, no dedicated leased lines, handled by two firewalls. What's this deployment called, name a protocol used for it, and explain why calling the firewall "Layer 3 because it routes" is still wrong even in this VPN context.**

<details>
<summary>Answer</summary>

**Site-to-site VPN** (as opposed to client-to-site/remote-access VPN, which connects a single user in). **IPsec**, in tunnel mode, is the standard for this — it encrypts the entire IP packet. The firewall isn't routing when it builds this tunnel: routing means making a forwarding decision based on destination IP and a routing table, while VPN tunneling means encapsulating and encrypting traffic and sending it across a path that's already routed — the public internet. A firewall is associated with Layer 3/4 because of what it filters on (IP addresses and ports), not because a VPN feature happens to touch IP packets. Reading or modifying IP headers doesn't make something a router.

</details>

**5. Two workloads: a shared file server for 50 employees, and a high-transaction SQL database needing low latency and high IOPS. Which storage fits each — NAS or SAN — and what's the real difference between file-level and block-level access?**

<details>
<summary>Answer</summary>

**NAS** for the file server, **SAN** for the database. The real distinction isn't "how much data you fetch" — it's **who owns the filesystem**. With NAS, the client asks for a file by name over a file-sharing protocol (SMB/NFS), and the NAS's own OS manages the actual filesystem and disk blocks underneath. With SAN, the client gets direct access to raw storage blocks — the client's own OS builds and manages the filesystem on top, so it looks and performs like a locally attached disk. That's why SAN is faster: there's no file-protocol layer in the way. Common SAN protocol: **iSCSI** (block storage over standard Ethernet/TCP-IP).

</details>

**6. A laptop can't get an IP address and is stuck on something like 169.254.x.x. What's normally responsible for assigning IPs, what's that specific fallback address range called, and what protocol/port does the process use?**

<details>
<summary>Answer</summary>

A **DHCP server** — often built into the router on small networks, but on larger networks frequently a separate dedicated server, with the router only doing **DHCP relay** across subnet boundaries, since DHCP requests are broadcast traffic that doesn't cross subnets on its own. `169.254.x.x` is called **APIPA** (Automatic Private IP Addressing) / link-local addressing — the device self-assigns it when it can't reach a DHCP server, which lets it talk to others on the same local segment but with no default gateway, so nothing off-subnet works. DHCP runs over **UDP**, port **67** (server) and port **68** (client).

</details>

**7. Traffic needs to be filtered based on the actual application/content being sent — not just IP or port — like blocking specific sites or catching malware signatures inside otherwise-allowed traffic. Which device, what's the general term for the technique it uses, and why can't a traditional stateful firewall do this on its own?**

<details>
<summary>Answer</summary>

An **NGFW** (Next-Generation Firewall). The technique it uses is called **deep packet inspection (DPI)** — actually looking inside the packet payload rather than just matching headers. A traditional stateful firewall can't do this because "stateful" only means it tracks connection state — recognizing a packet as part of an already-established session — which is a separate capability from inspecting application-layer content. Traditional firewalls filter at Layer 3/4 (IP/port); NGFWs extend that up to Layer 7.

</details>

**8. Explain the difference between IDS and IPS in terms of where they physically sit in the traffic path — not just what they do — and why that placement is the direct cause of the alert-vs-block difference.**

<details>
<summary>Answer</summary>

**IDS** sits **out-of-band** — it receives a mirrored copy of traffic, typically via a SPAN port or network tap. Because it's only ever looking at a copy, it has no ability to touch or stop the original packet — all it can do is alert. **IPS** sits **inline**, directly in the traffic path, so every packet has to pass through it — which is what lets it inspect and decide, in real time, whether to forward or drop the packet before it reaches the rest of the network. The alert-vs-block difference isn't an arbitrary feature choice, it's a structural consequence of placement: an out-of-band device physically cannot intervene, an inline device structurally cannot just watch. Trade-off: because IPS is inline, it's also a potential point of latency or failure.

</details>

**9. A load balancer spreads traffic across three web servers. A user logs in, and for the rest of their session all their requests need to keep hitting the same server, because their session data is stored locally there. What's this concept called, and name a common method used to achieve it?**

<details>
<summary>Answer</summary>

**Session persistence** (also called session affinity or sticky sessions). Common methods: **source IP affinity/hashing** — mapping a client's source IP consistently to the same backend server, simple but unreliable if many users share one public IP behind NAT — and **cookie-based persistence**, where the load balancer inserts a cookie identifying which server the client was assigned and reads it on every subsequent request. Cookie-based is generally the more common and reliable method.

</details>

**10. Explain the difference between a proxy server and a reverse proxy — specifically who each one acts on behalf of — and give one practical use case for each.**

<details>
<summary>Answer</summary>

A **forward proxy** (the standard meaning of "proxy server") sits in front of clients and acts **on behalf of the client** — the destination server sees the proxy's IP, not the real client's. Use case: a company filtering, logging, or caching what its employees access on the outbound internet. A **reverse proxy** sits in front of servers and acts **on behalf of the server** — the client believes it's talking directly to the real server and has no visibility into what's actually behind it. Use case: hiding backend servers from direct exposure, load balancing across them, or SSL termination. One-line way to keep it straight: **forward proxy hides the client, reverse proxy hides the server.**

</details>

---

## Mistakes I made in this quiz

Keeping this section honest, not cleaned up, since re-reading my own wrong answers is more useful than re-reading correct ones.

- Blamed the **switch** for a "same subnet works, cross-subnet fails" symptom, because I latched onto "MAC address, Layer 2" instead of reasoning through the actual symptom. Should have been the router/default gateway — a switch failure wouldn't selectively break only cross-subnet traffic.
- Knew a router was needed for inter-VLAN communication but skipped the actual reasoning (VLANs are separate broadcast domains, a switch has no Layer 3 concept) and didn't have the term **router-on-a-stick** ready.
- Blanked on **fat AP vs. thin AP** even after being handed the exact terms to use in the question.
- Said a VPN-capable firewall is "Layer 3 because it also routes the data" — mixed up **VPN tunneling** (encryption/encapsulation over an already-existing path) with actual **routing** (forwarding decisions via a routing table). Two unrelated capabilities.
- Described block-level access as "you don't have to fetch the whole file" — that's the effect, not the mechanism. The real distinction is **who owns the filesystem**, not how much data moves.
- Said "the router is responsible for assigning IPs" without naming DHCP properly, and didn't know `169.254.x.x` is **APIPA**, or that DHCP runs on **UDP 67/68**, at all.
- Could describe *what* IDS and IPS each do (alert vs. block) but not *where* each one sits (out-of-band vs. inline) or why that placement is what actually causes the capability difference.
- Never heard the term **session persistence** for the load balancer scenario — understood the behavior needed, had no name and no method (cookie/source IP) for it.
- Completely swapped **forward proxy and reverse proxy** — said the proxy acts for the server and the reverse proxy acts for the client. Backwards both ways.
