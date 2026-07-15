# Introduction to IP — Quiz

Questions I got asked on this topic, written down so I can re-test myself later instead of just re-reading the note and assuming I remember it.

## How to use this

Don't scroll straight down. Read a question, answer it out loud like I'm in an interview, then open the answer and check. If I got it half right, that still counts as wrong — the point of this file is catching the gap between "I know the shape of it" and "I can say it precisely."

---

## Round 1 — Scenarios

**1. You're looking at a single Ethernet frame in a packet capture. Walk through the order of headers from outside to inside, to get to the actual HTTP data — and say what's sitting inside the IP payload before you even reach TCP.**

<details>
<summary>Answer</summary>

Order is: **Ethernet header → IP header → TCP or UDP header → application data (HTTP etc.) → Ethernet trailer.**

Ethernet is the outermost layer, not IP — it wraps the whole thing, and there's an Ethernet trailer at the very end too, not just a header at the start. Inside the IP header comes the IP payload, and that payload is the TCP or UDP header plus whatever's inside that (the actual app data).

Also — say "frame" and "packet" correctly. A **frame** is the whole thing: Ethernet header + IP packet + Ethernet trailer. A **packet** is what's inside: IP header + payload. And the thing carrying port numbers is the **TCP or UDP header**, not "a data packet" — that's not a real term.

</details>

**2. Someone says "UDP traffic drops under load, but TCP on the same link is fine." Is that surprising? Explain using the actual mechanism difference — not just "TCP is reliable, UDP isn't."**

<details>
<summary>Answer</summary>

Not surprising — it's expected, because of **flow control**, not just reliability.

TCP has an ongoing back-and-forth between sender and receiver. That conversation lets the **receiver tell the sender to slow down or speed up** before things get overwhelmed. UDP has no session and no feedback loop at all, so nothing ever tells the sender to back off — it just keeps sending at the same rate no matter what's happening on the other end. That's why UDP is the one that drops under load.

One thing to not mix up: "unreliable" doesn't mean UDP has a worse chance of a packet arriving. TCP and UDP have the exact same odds of getting a packet across the network. The difference is UDP gives zero acknowledgment, so there's no way to *know* if it arrived or recover if it didn't.

</details>

**3. Your team wants to run an app on TCP 8443 instead of 443, claiming it adds security through obscurity. Do you agree? Also — can you run a UDP service on port 8443 at the same time with zero conflict?**

<details>
<summary>Answer</summary>

Don't agree on the security point — changing the port number is not a security control. It's just an addressing detail. The firewall is what actually decides allow/deny, and a random port number does nothing to increase or decrease real security.

On the second part — yes, no conflict at all. **TCP ports and UDP ports are two separate, independent namespaces.** A socket is IP address + protocol + port number, so TCP 8443 and UDP 8443 are two completely different sockets that just happen to share a number. Same logic as DNS running on TCP/53 and UDP/53 on the same box at the same time.

Easiest way to keep this straight: think of TCP and UDP as two separate buildings, both of which happen to have an apartment numbered 8443. Different buildings, no clash. The actual restriction is: you can't have two different services both trying to listen on the **same protocol + same port + same IP** at once — same building, same apartment, one tenant only.

</details>

**My follow-up doubt on this one — "I still don't get it — how can TCP and UDP run on the same IP and same port at the same time, but not two other services? Explain this in easy English."**

<details>
<summary>Answer</summary>

Think of it like two separate buildings, not one.

TCP and UDP are not the same system sharing space. They're like two separate apartment buildings that both happen to have an "apartment number 22." Just because Building A has apartment 22 and Building B also has apartment 22 doesn't mean there's a conflict — they're in different buildings. Nobody gets confused about which "22" you mean, because you always say which building too.

So: TCP port 22 = apartment 22, in the TCP building. UDP port 22 = apartment 22, in the UDP building. Two totally different addresses that just happen to use the same number. A computer can have SSH living in "TCP apartment 22" and some other service living in "UDP apartment 22" at the same time — no clash, because they're not actually in the same building.

Now the part that IS restricted: inside one building (say, the TCP building), only one tenant per apartment number. So you can't have two different services both trying to sit in "TCP apartment 22" — like SSH and some other TCP program both trying to listen on TCP port 22 on the same IP. Whoever shows up second gets rejected.

That's the whole rule: same building (same protocol), same apartment (same port), same address (same IP) → only one tenant allowed. Different buildings (TCP vs UDP) → totally separate, apartment 22 can exist in both with zero problem.

</details>

**My follow-up doubt — "So how do I know which services use the same protocol and which use a different one?"**

<details>
<summary>Answer</summary>

A few ways to actually know this in practice:

**1. The packet itself tells you.** Inside the IP header there's a field literally called "Protocol." It's a number that says whether the payload is TCP (value 6) or UDP (value 17). Tools like Wireshark read that field for you and just label the packet "TCP" or "UDP" directly — no guessing needed, it's explicitly marked in every packet.

**2. Well-known services are documented — you learn/memorize the common ones.** Certain applications default to a specific protocol: HTTP → TCP 80, HTTPS → TCP 443, SSH → TCP 22, DNS → mostly UDP 53 (but also TCP 53 for larger responses like zone transfers), VoIP/RTP-style traffic → typically UDP (no time to wait for retransmits on a live call). This isn't something you derive logically — it's assigned by IANA and used by convention. Servers pick TCP or UDP based on whether the app needs reliability (TCP) or speed with no retransmission delay (UDP).

**3. On a live machine, tools show you directly.** Commands like `netstat -tulnp` or `ss -tulnp` on Linux list every listening service along with whether it's bound as TCP or UDP. You're not guessing — the OS reports it.

So in short: you don't figure out the protocol from the port number alone — the number by itself doesn't tell you. You either read it off the packet (protocol field), look it up against known service assignments, or check the OS directly.

</details>

**4. Packet capture shows source port 51422, destination port 22. Which side is client, which is server, how do you know, and what does the number range tell you about each port?**

<details>
<summary>Answer</summary>

51422 is the client, 22 is the server (SSH specifically). Reasoning: well-known ports run **0–1023**, and 22 falls in that range. Ephemeral (temporary, client-side) ports run **1024–65535** — not 65555, the field is 16-bit so the real ceiling is 65535 (2^16 − 1).

Important caveat: this is reading a **convention**, not a rule the protocol enforces. Nothing stops a server from binding to a high port or a client from using a low one — it's just what's normally expected, so you'd phrase it in an interview as "based on convention" rather than stating it like it's guaranteed.

</details>

**5. A capture shows SYN, then SYN-ACK, then ACK, then data starts flowing. What protocol, and what does this map to specifically? Could this pattern ever show up in UDP?**

<details>
<summary>Answer</summary>

This is TCP, and it's the **three-way handshake** — it's what makes TCP **connection-oriented**. It's the formal setup process before any application data moves, and there's a matching formal teardown when the session ends.

Big thing to not mix up: the handshake is **not** the same thing as TCP's reliability mechanism (ACKs per data segment, retransmission, resequencing lost data). Those are two separate TCP features. The handshake happens *before* data flows and just negotiates "are we ready to talk" — it doesn't move or recover any application data itself. Reliability is a different mechanism that runs *during* data transfer.

This would never happen in UDP because UDP is **connectionless** — there's no setup and no teardown process of any kind. The missing ACKs are a side effect of that design choice, not the actual reason the handshake doesn't exist.

</details>

**6. A client is talking to a server running web, VoIP, and email at the same time, and all that traffic arrives mixed together. How does the client's OS actually sort out which incoming packet belongs to which app? What fields is it using?**

<details>
<summary>Answer</summary>

The OS keeps a table of every open socket, each one identified by **protocol + local IP + local port + remote IP + remote port**. When a packet comes in, the OS checks those same fields against the table to find the match.

The key detail: each application opened its own connection using a different **ephemeral source port** on the client side. So even though all three replies are coming from the same server IP, each one arrives addressed to a different port on the client — because that "destination port" on the incoming reply is actually the client's own ephemeral port from when it started that connection. That's the real mechanism, not just "port and protocol help the OS classify traffic" in the abstract.

</details>

**7. "We should block port 443 outbound — that stops this app from reaching the internet no matter what." Bulletproof control, or not — and why, based on what we know about ports?**

<details>
<summary>Answer</summary>

Not bulletproof. Port numbers are convention, not enforcement — nothing forces an app to use 443 for HTTPS. A developer can just point the app at a different port (8443, 9999, whatever) and traffic still works exactly the same at the protocol level, so the block only stops the one conventional door, not the capability.

Deeper point: port-based blocking filters on **assumed intent**, not actual behavior. The firewall is trusting "port 443 = HTTPS," but nothing guarantees that's true. This is why real security depends on firewall policy and deeper inspection, not just matching a port number — the port itself carries zero security weight on its own.

</details>

**8. TCP guarantees reliable delivery through ACKs and retransmission. Does that mean TCP guarantees fast or low-latency delivery? Why does real-time voice/video almost always choose UDP over TCP even though TCP is "more reliable"?**

<details>
<summary>Answer</summary>

No — reliable and fast are two different guarantees entirely. TCP promises data arrives complete and in order, not that it arrives quickly.

The actual latency cost is called **head-of-line blocking**: TCP enforces in-order delivery to the application, so if a packet is lost, any packets that arrived *after* it get held back in a buffer and withheld from the app until the missing one is retransmitted and fills the gap. The sender isn't "stopping" — the wire keeps carrying data — it's specifically the delivery-to-application step that stalls.

UDP has no ordering guarantee and no retransmission, so packets get handed to the app immediately, gaps and all. For live voice/video, a late retransmitted packet is useless anyway — you can't play a delayed frame in the past — so those apps would rather drop a bad frame and keep moving than pause for perfect data.

</details>

**9. Using the truck/box/house/room analogy — what does the destination IP address correspond to? And why isn't a port number alone, without an IP address, enough to deliver data anywhere on the internet?**

<details>
<summary>Answer</summary>

Destination IP = the house address — what routers actually use to move the packet across the network to the right device.

A port alone can't deliver anything because it's only **locally meaningful, not globally routable**. Port 443 by itself is ambiguous — millions of devices are potentially running something on port 443 right now. Routers move traffic purely based on IP address; routing tables don't even look at port numbers. The port only gets read *after* the packet has already reached the correct device — it's the last step, used to hand the data to the right local application, not to get it across the network in the first place.

</details>

**10. Capture shows IP protocol field = 17, source port 5004, destination port 60112, no SYN/ACK anywhere, traffic keeps flowing despite visible gaps in sequence numbers. What protocol, what's the evidence, and what real-world app is this likely to be?**

<details>
<summary>Answer</summary>

This is UDP. The strongest, most direct evidence is the **IP protocol field = 17** — that field exists specifically to state TCP (6) or UDP (17) explicitly in the IP header, no inference needed, tools like Wireshark just read it. The missing handshake and the continued flow despite gaps are supporting evidence (connectionless, no retransmission), but they're circumstantial compared to the protocol field itself — always lead with the definitive evidence, not the secondary symptoms.

Port 5004 is a direct match to the well-known VoIP port from the material's own example, so this is almost certainly real-time voice traffic — an app that would rather drop a bad audio packet than stall waiting for a retransmit, since a "recovered" late packet is worthless once the moment's passed.

</details>

---

## Mistakes I made in this quiz

Keeping this section honest, not cleaned up, since re-reading my own wrong answers is more useful than re-reading correct ones.

- Skipped the **Ethernet header and trailer** entirely and started my header-order answer at IP instead — called a frame "a packet" and called the TCP/UDP header "a data packet," which aren't real terms.
- Only explained TCP's ack/retransmission on the UDP-drops-under-load question and missed **flow control**, which is the actual mechanism that answers "why does UDP drop and TCP doesn't."
- Thought **TCP and UDP couldn't use the same port number at the same time** — they can, because protocol is part of the socket tuple. Only same-protocol + same-port + same-IP is restricted.
- Said the ephemeral port range was **1024–65555** instead of **65535**, and didn't flag that "51422 = client" is a convention, not a guarantee.
- Mapped the **three-way handshake to reliability/retransmission** instead of connection setup — conflated two separate TCP properties (session management vs. delivery guarantee) into one answer.
- On the multiplexing question, said "port numbers and protocol help the OS classify traffic" without naming the actual mechanism — the socket table match on protocol + local/remote IP + local/remote port, and the role of ephemeral source ports.
- Said TCP "stops the data flow" when a packet is missed, instead of naming **head-of-line blocking** — the sender doesn't stop, the receiver withholds already-arrived data from the app while waiting on the gap.
- On the IP-vs-port analogy question, said a port "doesn't have the capability to move over network" instead of the actual reason — a port is only locally meaningful and isn't globally routable, so routers don't even look at it.
- On protocol identification from a capture, led with the weaker circumstantial evidence (no SYN/ACK, missing packets) instead of citing the **IP protocol field (6 = TCP, 17 = UDP)** first, which is the direct, unambiguous answer.
