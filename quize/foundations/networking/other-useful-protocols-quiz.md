# Other Useful Protocols (ICMP, GRE, IPSec) — Quiz

Questions I got asked on this topic, written down so I can re-test myself later instead of just re-reading the note and assuming I remember it.

## How to use this

Don't scroll straight down. Read a question, answer it out loud like I'm in an interview, then open the answer and check. If I got it half right, that still counts as wrong — the point of this file is catching the gap between "I know the shape of it" and "I can say it precisely."

---

## Round 1 — Scenarios

**1. You ping a host and get nothing back, but you also get an ICMP message saying the TTL expired in transit. What's actually happening on the network, and what tool would you reach for to see exactly where this occurs?**

<details>
<summary>Answer</summary>

Every IP packet carries a **TTL** field that gets decremented by 1 at each router hop. When TTL hits 0, that router drops the packet and sends back an **ICMP Time Exceeded** message to the sender.

The right tool is **traceroute** (`tracert` on Windows) — not the route table. A route table only shows your own device's next hop, not the full path or where things are actually breaking down. Traceroute deliberately exploits TTL expiry: it sends packets with TTL=1, then 2, then 3, and so on, so each router along the path replies in turn, revealing the full hop-by-hop route.

Common trap: TTL expiring isn't automatically a problem — it's also just normal traceroute behavior. It's only a real issue if a normal ping to the final destination times out, or traceroute shows the path dying at a hop and never reaching the destination.

</details>

**2. Your team is designing a site-to-site connection between two offices. Someone suggests "just use GRE, it's simple." What's your pushback, and how would you fix the design?**

<details>
<summary>Answer</summary>

**GRE tunnels, it doesn't protect.** It encapsulates the original packet and sends it across the tunnel, but provides **no encryption**. If GRE is the only thing in place, all data crossing the public internet between the two sites is in plain text — readable by an ISP, a compromised router, or anyone doing a man-in-the-middle capture.

The fix: layer **IPSec on top of GRE** (GRE handles flexible tunneling — including non-IP or private-addressed traffic — while IPSec handles encryption), or skip GRE entirely and use a direct IPSec tunnel if you don't need GRE's extra flexibility (like carrying multicast or routing protocol traffic).

Lead with the risk, not a clarifying question — state "this is plaintext until proven otherwise," then ask if IPSec is layered underneath.

</details>

**3. You're troubleshooting an IPSec tunnel between a Cisco firewall on one end and a Palo Alto on the other. Phase 1 completes but Phase 2 fails. Explain what's different about what each phase negotiates, and give a plausible reason Phase 2 specifically would fail even though Phase 1 succeeded.**

<details>
<summary>Answer</summary>

**Phase 1 (ISAKMP)** — runs over **UDP port 500**, uses Diffie-Hellman to build a shared secret key. This is about authenticating both sides and building a secure channel to negotiate *in* — it doesn't protect any real data yet.

**Phase 2** — negotiates the actual data-protection parameters: encryption cipher, key sizes, and the inbound/outbound Security Association. This builds the ESP tunnel that carries real traffic.

**Why Phase 1 succeeding doesn't guarantee Phase 2 will:** they negotiate completely separate parameter sets. Phase 1 succeeding only proves both devices can authenticate each other. Phase 2 has its own list of proposals — cipher, hashing, PFS group, and critically, **proxy IDs / interesting traffic** (which subnets are allowed through the tunnel). Most common real-world cause: mismatched Phase 2 ciphers, or mismatched proxy-ID/subnet definitions between the two vendors' configs.

</details>

**4. A security auditor flags that your VPN tunnel is using AH instead of ESP, and says this is a confidentiality risk. Is the auditor right? Why or why not?**

<details>
<summary>Answer</summary>

**Yes, the auditor is right.**

**AH** (Authentication Header) validates the data you're receiving — integrity and authentication — but sends everything **in the clear**. No encryption at all.

**ESP** (Encapsulating Security Payload) encrypts the original data and the trailer, and still provides an integrity check — so it covers confidentiality *and* integrity/authentication in one protocol.

If a tunnel is using AH only, anyone capturing that traffic can read the payload in plain text — they just can't tamper with it undetected. The fix is to switch to ESP, which is what the vast majority of real-world IPSec deployments use, precisely because AH's lack of encryption makes it fairly niche today.

Worth knowing: AH and ESP *can* technically be combined, but almost nobody does this in practice — ESP alone already covers integrity, so stacking AH on top is mostly redundant.

</details>

**5. You capture packets off a WAN link and can clearly read the source and destination IP addresses of an IPSec-protected conversation, even though the payload is encrypted. What mode is this tunnel using, and what would you change to hide the original IPs too?**

<details>
<summary>Answer</summary>

**Transport mode.** In transport mode, an IPSec header gets inserted between the IP header and the data, but the **original IP header stays in the clear**. Only the data portion gets encrypted (with ESP) and an IPSec trailer gets added. So the payload is protected, but source/destination IPs are still readable.

The fix: switch to **tunnel mode**. In tunnel mode, the original IP header *and* the data are both encrypted, and a brand new IP header is added on the outside — showing only the two IPSec endpoints (the firewalls/concentrators), not the original hosts. This is why tunnel mode is the default for almost every real-world site-to-site VPN — it hides internal topology entirely from anyone sniffing the WAN link.

</details>

**6. From an AppSec/design-review lens: you see "Site A —— GRE Tunnel —— Site B" on an architecture diagram with a note that says "VPN established." No other detail. What's your finding, and what do you ask the engineer who built this?**

<details>
<summary>Answer</summary>

**Finding:** GRE by itself doesn't encrypt anything — it only encapsulates. The label "VPN established" is misleading on its own; this diagram doesn't confirm whether data is actually protected in transit.

**Follow-up question:** Is IPSec running underneath this GRE tunnel, or is GRE the only thing here? If it's GRE alone, this traffic is crossing the public internet in plaintext and needs to be flagged regardless of what the diagram calls it.

State the finding first, *then* ask the clarifying question — don't just ask and stop there. A security review deliverable leads with the risk.

</details>

**7. During a vendor security questionnaire review, a third party says: "All data in transit is protected via IPSec." That's the entire sentence — no other detail. Is that enough to sign off? What's your follow-up question?**

<details>
<summary>Answer</summary>

**Not enough to sign off.** "IPSec" alone doesn't tell you whether AH or ESP is in use — and that distinction is the difference between data being encrypted or just integrity-checked in the clear.

**Follow-up:** "Is this AH, ESP, or both?" If it's AH-only, the vendor's data is not actually confidential in transit, regardless of the "protected via IPSec" claim. Vague protocol names without the "which sub-protocol/mode" detail are one of the most common ways vendor security answers hide real gaps — this is worth blocking sign-off over until answered.

</details>

**8. From a STRIDE lens: engineers are connecting to internal infra over IPSec in transport mode "because it's faster and simpler." What risk are you flagging, and is the tradeoff reasonable?**

<details>
<summary>Answer</summary>

**Risk: Information Disclosure of network metadata/topology** — not plaintext data. Even with ESP encrypting the payload in transport mode, the original source/destination IP addresses stay visible. Anyone sniffing the link sees exactly which internal hosts engineers are connecting to — reconnaissance value for a follow-on attack.

**On the tradeoff:** transport mode is slightly more efficient (no extra IP header), but for remote access into internal infra, you're trading a small overhead savings for exposing your internal topology. Usually not a reasonable tradeoff unless there's a genuine, measured performance constraint — worth pushing back on in review.

Common mix-up to avoid: this is **not** "data travels in plaintext." The payload is still encrypted by ESP — what leaks is the header/metadata, not the content.

</details>

---

## Round 2 — Quick-fire recall

Same rules — say the answer out loud before opening it. No context given on purpose; this round tests raw recall under pressure, since that's where the gaps actually show up.

<details>
<summary>1. What protocol does ping use?</summary>ICMP</details>
<details>
<summary>2. Is ICMP carried over TCP or UDP?</summary>Neither — it's its own protocol, carried directly by IP</details>
<details>
<summary>3. What ICMP message tells you a packet's TTL hit zero?</summary>Time Exceeded</details>
<details>
<summary>4. What tool deliberately sends packets with increasing TTL values to map a path?</summary>Traceroute (tracert)</details>
<details>
<summary>5. Does GRE provide encryption on its own?</summary>No — encapsulation/tunneling only, no protection</details>
<details>
<summary>6. What do you need to add to GRE to actually secure the data?</summary>IPSec (or another VPN/encryption protocol)</details>
<details>
<summary>7. What does IPSec stand for?</summary>Internet Protocol Security</details>
<details>
<summary>8. Which IPSec sub-protocol gives integrity + authentication but NOT confidentiality?</summary>AH (Authentication Header)</details>
<details>
<summary>9. Which IPSec sub-protocol gives integrity + authentication + confidentiality?</summary>ESP (Encapsulating Security Payload)</details>
<details>
<summary>10. What does IKE stand for, and what does it produce?</summary>Internet Key Exchange — produces a Security Association (SA)</details>
<details>
<summary>11. What port and transport does IKE Phase 1 (ISAKMP) use?</summary>UDP port 500</details>
<details>
<summary>12. What key exchange algorithm is commonly used in Phase 1?</summary>Diffie-Hellman</details>
<details>
<summary>13. What does Phase 1 actually establish?</summary>Mutual authentication + a secure channel to negotiate in (no real data protected yet)</details>
<details>
<summary>14. What does Phase 2 negotiate?</summary>Ciphers, key sizes, and inbound/outbound Security Associations for the actual IPSec/ESP tunnel</details>
<details>
<summary>15. In transport mode, is the original IP header encrypted?</summary>No — it stays visible; only the data/payload is encrypted</details>
<details>
<summary>16. In tunnel mode, is the original IP header encrypted?</summary>Yes — original header + data both encrypted, wrapped in a new outer IP header</details>
<details>
<summary>17. Which mode is the default for most real-world site-to-site VPNs?</summary>Tunnel mode</details>
<details>
<summary>18. What does a VPN concentrator do, and where is it often built in?</summary>Handles encryption/decryption at a central point — often built into an existing firewall</details>

**Known trip-up points — drill these specifically:**
- Transport mode does **not** mean plaintext data — the payload is still encrypted by ESP. What's exposed is the original IP header (source/destination), not the content.
- Route table ≠ traceroute. Route table shows only your own next hop; traceroute reveals the full path by exploiting TTL expiry.
- Phase 1 succeeding says nothing about Phase 2 — they negotiate independent parameter sets (auth/channel setup vs. actual cipher/subnet agreement) and can fail independently.
- AH = integrity only, no encryption. Don't assume "IPSec" alone implies confidentiality — always confirm AH vs ESP.
- GRE ≠ VPN. A "GRE tunnel" label on a diagram says nothing about whether the data is actually encrypted.
