# Networking Functions — Quiz

Questions I got asked on this topic, written down so I can re-test myself later instead of just re-reading the note and assuming I remember it.

## How to use this

Don't scroll straight down. Read a question, answer it out loud like I'm in an interview, then open the answer and check. If I got it half right, that still counts as wrong — the point of this file is catching the gap between "I know the shape of it" and "I can say it precisely."

---

## Round 1 — Scenarios

**1. Walk through exactly what happens, hop by hop, when a packet's TTL hits 0. Who drops it, and what — if anything — gets sent back to the sender?**

<details>
<summary>Answer</summary>

The router that receives the packet with TTL=1 decrements it to 0 and **drops it right there** — it never forwards a TTL=0 packet onward, and the payload/content goes with it since packets aren't split. The part that's easy to miss: the router doesn't drop it silently. It sends an **ICMP "Time Exceeded"** message back to the original source IP. This ICMP reply is the entire mechanism `traceroute`/`tracert` is built on — sending packets with TTL=1, 2, 3... and using the "time exceeded" replies to map each hop along the path.

</details>

**2. A junior engineer says "CDN is basically just caching." Push back on that — what else does a modern CDN typically do?**

<details>
<summary>Answer</summary>

Caching is the visible benefit, but a modern CDN also does: **DDoS mitigation** (distributed edge nodes absorb and filter volumetric attacks before they hit origin), **SSL/TLS termination** (edge servers handle the encryption handshake, offloading CPU work from origin), **load balancing** across edge nodes (routing users to the nearest/healthiest node, often via Anycast or geo-DNS), and **origin shielding** (most requests never reach the actual origin server at all). Some modern CDNs also run actual **edge compute** (Cloudflare Workers, Lambda@Edge), not just serving static cached files. "Just caching" undersells the security and availability value, which is usually the real reason enterprises pay for it.

</details>

**3. What's the practical difference between a site-to-site VPN and a remote-access (client) VPN, and where does the concentrator sit differently in each?**

<details>
<summary>Answer</summary>

**Site-to-site VPN**: connects two gateways/firewalls at two *networks*, tunnel is always-on and transparent to end users — no client software installed on individual devices.

**Remote-access (client) VPN**: the user installs **client software** (Cisco AnyConnect, GlobalProtect, OpenVPN client, etc.) on their own device — laptop or phone. That client connects out to a **VPN concentrator/headend sitting at the corporate network edge**, which is where the server-side hardware or software actually lives — not on the user's end.

The concentrator is always on the organization's side in both cases. The real difference is what's on the *other* end of the tunnel: another gateway (site-to-site) vs. a single client device (remote-access).

</details>

**4. You're asked to prioritize VoIP traffic over bulk file transfers using QoS. What's actually happening under the hood — what gets marked, and what mechanism enforces the priority at the router/switch?**

<details>
<summary>Answer</summary>

**What gets marked**: packets get tagged with a priority value in their header — the common field is **DSCP (Differentiated Services Code Point)**, a 6-bit field in the IP header. Voice traffic typically gets marked with a DSCP value like **EF (Expedited Forwarding)**, signaling "treat this as high priority" to every device downstream.

**What enforces it**: the router/switch maintains multiple **queues** instead of one FIFO line. Packets get sorted into queues based on their marking, and a **scheduler** services the high-priority queue first or more often (e.g. Low Latency Queuing, Weighted Fair Queuing). Two related enforcement tools: **traffic shaping** (delays/buffers excess traffic to smooth it to a target rate, doesn't drop) and **traffic policing** (drops or re-marks traffic exceeding a rate limit, no buffering).

Flow to remember: **classify → mark (DSCP) → queue → schedule.**

</details>

**5. Why doesn't a routing loop take down the whole network permanently, even on a badly misconfigured static route?**

<details>
<summary>Answer</summary>

Because of **TTL**. Every time a looping packet passes through a router, its TTL decrements by 1 — same mechanism as the packet-drop question above. Eventually TTL hits 0, the router drops the packet, and sends an ICMP Time Exceeded reply. So the loop is self-limiting per packet — a given packet can only bounce back and forth 64 or 128 times (depending on the OS default) before it's forcibly killed. Without TTL, a routing loop would actually be catastrophic — packets would loop forever with no cap, degrading the network toward total congestion collapse. Note: TTL doesn't fix the loop itself or stop new packets from entering it — it only guarantees no single packet lives forever while the misconfiguration is being fixed.

</details>

**6. If a network has no QoS configured at all, what actually happens to different types of traffic under congestion — is it just "first come first served," or is something else going on by default?**

<details>
<summary>Answer</summary>

Yes — the default is genuinely **best-effort delivery** through a single **FIFO (First In, First Out)** queue. No traffic gets special treatment. When that single queue fills up under congestion, the router does **tail drop** — it discards whatever new packets show up once the queue is full, regardless of what they are. This is where TCP and UDP diverge: **TCP** notices the drop and triggers congestion control, backing off its send rate. **UDP** (voice, video) has no such mechanism — it keeps sending at the same rate, so the user experiences it as jitter, choppy audio, or dropped frames. That's precisely why QoS matters specifically for real-time traffic — without it, a big file download can starve a call, not because anyone configured it that way, but because FIFO + tail drop doesn't discriminate.

</details>

**7. A VPN "encrypts the communication," but encryption alone doesn't guarantee the data hasn't been tampered with in transit. What's the piece of the puzzle that ensures integrity (not just confidentiality) of VPN traffic?**

<details>
<summary>Answer</summary>

**HMAC (Hash-based Message Authentication Code)**. The sender runs the packet's data through a hash function (like SHA-256) combined with a shared secret key, producing a fixed-length digest attached to the packet. The receiver does the same calculation with the same shared key and compares results — if even one bit was altered in transit, the hashes won't match, and tampering is detected. This is a separate operation from encryption: encryption handles **confidentiality**, HMAC handles **integrity**. In IPsec specifically: **AH (Authentication Header)** handles integrity/authentication only, **ESP (Encapsulating Security Payload)** handles both encryption and integrity — most VPNs use ESP since it covers both.

</details>

**8. If a user requests content that isn't cached yet at their nearest CDN edge server (a "cache miss"), does the request just fail, or is there a fallback?**

<details>
<summary>Answer</summary>

It doesn't fail — there's a fallback chain, often called **origin pull**. The request hits the nearest edge node (cache hit → served immediately). On a **cache miss**, it may check a mid-tier/regional cache if the CDN has one, and if still a miss, the request goes all the way back to the **origin server**. The origin serves the content, and the edge node **stores a copy locally on the way back**, so the *next* user requesting that content gets a cache hit. A cache miss is slower (comparable to a non-CDN request) but self-healing — the CDN "learns" popular content over time, which is why CDNs handle sudden traffic spikes (viral posts, product launches) well: only the first few requests trickle to origin.

</details>

**9. If you see a reply packet arrive with a TTL of 51, what can you infer, and why can't you just say "the sender's OS has a TTL of 51"?**

<details>
<summary>Answer</summary>

You can't say the OS has TTL 51 because that number is what's **left after decrements**, not the starting value a device actually uses — real defaults only come from a small standard set (64 for Linux/macOS, 128 for Windows, 255 for some network gear). To estimate hop count, find the **smallest standard default that's still greater than 51** — that's 64 — and infer the packet traveled 64 − 51 = **13 hops**. This is an educated guess, not a certainty: if the packet actually started from 128 (Windows) with 77 hops in between, it would also show up as 51, and there'd be no way to distinguish the two cases from the number alone.

</details>

**10. Give one realistic scenario where two of these functions (VPN, QoS, CDN, TTL) genuinely interact or depend on each other — not just coexist in the same network.**

<details>
<summary>Answer</summary>

**VPN + QoS conflict.** When traffic goes through a VPN, the entire original packet (including its headers) gets encapsulated and encrypted inside a new outer packet. If a packet's DSCP marking lives on the *inner* (original) header, and a router along the path only inspects the *outer* (tunnel) header, it can't see the priority marking anymore — carefully prioritized VoIP traffic gets treated as generic best-effort once it enters the tunnel. Most VPN implementations fix this by **copying the DSCP value from the inner header to the outer header** during encapsulation specifically so QoS still works across the tunnel. If that copy doesn't happen (misconfigured or older gear), voice quality degrades over the VPN even though QoS looks "configured correctly" — because the priority marking never survives encryption/encapsulation.

</details>
