# Common Ports — Quiz

Questions I got asked on this topic, written down so I can re-test myself later instead of just re-reading the note and assuming I remember it.

## How to use this

Don't scroll straight down. Read a question, answer it out loud like I'm in an interview, then open the answer and check. If I got it half right, that still counts as wrong — the point of this file is catching the gap between "I know the shape of it" and "I can say it precisely."

---

## Round 1 — Scenarios

**1. A VoIP phone boots with zero configuration, grabs an IP via DHCP, then needs to pull its config file from a provisioning server. Which protocol and port handles that file pull, and why not just use FTP?**

<details>
<summary>Answer</summary>

**TFTP, UDP port 69.**

FTP brings overhead this scenario doesn't need — authentication, directory browsing, file rename/delete, a whole management layer — for a device that has *nothing* configured yet and just needs one file, fast, with no human involved. TFTP strips all of that away: no login, no directory ops, just "give me this file, quick." That's the actual trade-off being tested — simplicity/speed vs. features/security — not just the port number.

Also worth knowing: TFTP runs over **UDP**, not TCP — no handshake, no guaranteed delivery, which matches the "quick and dirty, no session needed" use case.

</details>

**2. Someone claims: "SSH and SFTP are unrelated protocols that just happen to share a port by coincidence." Correct them. Then explain why Telnet is effectively dead on modern networks, and give its port.**

<details>
<summary>Answer</summary>

Not a coincidence — **SFTP is SSH.** It doesn't have its own separate transport; it tunnels file transfer through the same encrypted SSH connection. That's exactly why they share **TCP port 22** — same underlying protocol doing the work, just used for a different purpose (one gives a remote shell, the other gives file transfer), both wrapped in SSH's encryption.

Telnet (**TCP 23**) does the identical job as SSH — remote terminal access — but sends everything, including login credentials, in **plaintext**. Anyone capturing packets on the network reads the login live. That's the whole reason it got replaced: same job, zero encryption.

</details>

**3. A mail server is configured to accept client mail submissions only on port 587, not port 25. Why would it do that intentionally — what's the actual difference between the two ports here?**

<details>
<summary>Answer</summary>

**Port 25 = server-to-server relay, plaintext by default. Port 587 = client-to-server submission, with TLS encryption and authentication.**

The direction matters: 587 is what a mail *client* (Outlook, a phone's mail app, etc.) uses to submit a message *to* its own outgoing mail server — not what one mail server uses to hand a message to another mail server. That stays on 25.

Common trap: assuming 587 exists mainly to stop spam. That's a side effect, not the design goal. The real reason 587 exists is to give client devices an encrypted, authenticated path for mail submission. Many ISPs blocking outbound port 25 on residential networks (because spambots abuse unauthenticated port 25) is a consequence layered on top — not what 587 was built for.

</details>

**4. A packet capture normally shows only UDP port 53 for DNS, but you suddenly see a burst of TCP port 53 traffic. What triggers that switch, and why does that specific scenario need TCP's reliability over UDP's speed?**

<details>
<summary>Answer</summary>

DNS switches to **TCP port 53** — same port number, different transport — for two situations: a **zone transfer** (one DNS server copying its entire zone database to another, e.g. primary → secondary), or a **response too large to fit in a single UDP datagram**.

Both need TCP because: zone transfers move a lot of data and need guaranteed, ordered delivery — you can't afford a dropped packet silently corrupting a database sync. And UDP has a practical size ceiling before fragmentation becomes a problem, while TCP handles large payloads cleanly through segmentation.

Don't write this as "TCP 531" or treat it as a different port — it's still 53, just carried over TCP instead of UDP.

</details>

**5. True or false: "SNMPv3 uses a different port than v1 and v2, because it's the secure version." Explain what actually changed between versions.**

<details>
<summary>Answer</summary>

**False.** All three SNMP versions use the **same ports** — UDP 161 for queries, UDP 162 for traps. The port never changes across versions. What changes is what happens *on* that port, not which port is used.

- **v1**: single query → single response, plaintext, no security
- **v2**: adds bulk transfers (query many variables at once), still plaintext
- **v3**: adds message integrity, authentication, and encryption — this is the real security upgrade

This is a common exam trap: "more secure version = different port" holds for some protocol pairs (HTTP 80 → HTTPS 443) but explicitly **not** for SNMP, where the security upgrade happens within the same port.

</details>

**6. A capture shows TCP port 445 on a Windows network. What's almost certainly happening, what functions does it cover beyond file sharing, and what's the legacy name for this same general capability?**

<details>
<summary>Answer</summary>

This is **SMB** (Server Message Block) traffic. It covers more than just file sharing — also **printer sharing** and **authentication**, all built directly into Windows Explorer with no extra software required.

Legacy angle: older Windows versions ran this over **NetBIOS** before modern Windows moved it to communicate directly over IP on TCP 445. You'll also see this functionality referred to as **CIFS** (Common Internet File System) — an alternate name, especially common in Linux/Samba contexts, not strictly a "legacy" version.

</details>

**7. Help desk needs to see a user's actual desktop — full GUI, not a command line — to troubleshoot visually. What protocol and port, and how is this fundamentally different from what SSH provides?**

<details>
<summary>Answer</summary>

**RDP, TCP port 3389.**

SSH gives an encrypted **command-line** session — you're typing commands, no visuals. RDP gives the **entire desktop** — mouse, windows, applications, the full visual experience, as if sitting at that machine.

Worth knowing: RDP-the-service is a Windows thing, but RDP *clients* exist for Mac, Linux, iOS, Android — so a Mac user can RDP into a Windows box, but you wouldn't typically see the reverse, since RDP as a listening service isn't a Mac/Linux-native thing.

</details>

**8. What is LDAP actually used for on a network, what port does it use unencrypted, and what's the real risk of capturing plaintext LDAP traffic — be specific about what it does and doesn't carry?**

<details>
<summary>Answer</summary>

LDAP (Lightweight Directory Access Protocol) queries a **hierarchical directory** of users and devices — organization → organizational units (e.g. Production, Support, Engineering) → individual objects (users like Jack or Daniel, or resources like a storage server). Port **TCP 389** unencrypted, **TCP 636 (LDAPS)** encrypted.

Important scope check: LDAP is a **directory service, not a file storage or transfer protocol** — it does not carry documents or files. What plaintext LDAP actually risks exposing:
- Usernames, job titles, department/OU structure
- Sometimes **authentication credentials**, when a device or app performs an LDAP "bind" (login) to the directory
- The full org hierarchy — reconnaissance value for an attacker planning a targeted attack

Don't overstate this as "any document or anything" leaking — the real risk is credential exposure during binds plus org-structure reconnaissance.

</details>

**9. Why does syslog use UDP instead of TCP, given that log data sounds important not to lose? Where does that traffic typically end up?**

<details>
<summary>Answer</summary>

Syslog uses **UDP port 514**. The reasoning is about scale: routers, switches, firewalls, and servers — potentially hundreds or thousands of devices — are all constantly generating log entries. If every single log message required a TCP handshake and acknowledgment, that overhead multiplied across every device and every log line would be enormous.

The trade-off is intentional: occasional message loss is acceptable in exchange for not overwhelming the network with connection overhead, especially since analysis usually depends on **patterns across huge volumes** of logs, not any single message arriving.

Destination: syslog traffic gets funneled into a **SIEM** (Security Information and Event Manager) — a centralized system that aggregates logs from every device so you can search and correlate across the whole network instead of checking each device individually.

</details>

**10. Rank these three by how much delivery/integrity guarantee they carry, and explain why: DNS queries (UDP 53), Syslog (UDP 514), FTP control channel (TCP 21).**

<details>
<summary>Answer</summary>

**FTP control (TCP 21) > DNS queries (UDP 53) > Syslog (UDP 514).**

**FTP control** — highest guarantee. TCP means handshake, acknowledgment, retransmission on drop. Makes sense — you don't want a "delete this file" or login command getting lost or arriving out of order.

**DNS queries** — middle. UDP, so no built-in guarantee, but the *application layer* compensates: if a response doesn't come back, the client just re-sends the query. Low stakes per packet (one lost query = one quick retry), so UDP's speed is worth it.

**Syslog** — lowest guarantee, and it's intentional — occasional message loss is acceptable given the volume of traffic and the fact that analysis leans on patterns across large datasets, not any single message.

The pattern worth internalizing: a protocol's port/transport choice usually reflects *why* it needs (or doesn't need) that guarantee — not an arbitrary design choice.

</details>

---

## Round 2 — Quick-fire port recall

Same rules — say the protocol/port out loud before opening the answer. No context given on purpose; this round is testing raw recall under pressure, since that's where the gaps actually show up.

<details>
<summary>1. FTP data channel</summary>TCP 20</details>
<details>
<summary>2. FTP control channel</summary>TCP 21</details>
<details>
<summary>3. SSH</summary>TCP 22</details>
<details>
<summary>4. SFTP</summary>TCP 22 (same port as SSH — it runs over SSH)</details>
<details>
<summary>5. Telnet</summary>TCP 23</details>
<details>
<summary>6. SMTP (server-to-server)</summary>TCP 25</details>
<details>
<summary>7. SMTP (client submission, TLS)</summary>TCP 587</details>
<details>
<summary>8. DNS (normal query)</summary>UDP 53</details>
<details>
<summary>9. DNS (zone transfer / oversized response)</summary>TCP 53 (same port as the query, different transport)</details>
<details>
<summary>10. DHCP (server)</summary>UDP 67</details>
<details>
<summary>11. DHCP (client)</summary>UDP 68</details>
<details>
<summary>12. TFTP</summary>UDP 69</details>
<details>
<summary>13. HTTP</summary>TCP 80</details>
<details>
<summary>14. HTTPS</summary>TCP 443</details>
<details>
<summary>15. NTP</summary>UDP 123</details>
<details>
<summary>16. SNMP (query)</summary>UDP 161</details>
<details>
<summary>17. SNMP (trap)</summary>UDP 162 — same port on v1, v2, and v3</details>
<details>
<summary>18. LDAP</summary>TCP 389</details>
<details>
<summary>19. LDAPS</summary>TCP 636</details>
<details>
<summary>20. SMB</summary>TCP 445</details>
<details>
<summary>21. Syslog</summary>UDP 514</details>
<details>
<summary>22. MS-SQL</summary>TCP 1433</details>
<details>
<summary>23. RDP</summary>TCP 3389</details>
<details>
<summary>24. SIP</summary>TCP 5060 / 5061</details>

**Known trip-up ports — drill these specifically:** Telnet (23, not a random high number), TFTP (69, not close variants like 61 or 96), DNS zone transfer (still 53, not a different number), SNMP across versions (still 161/162 on v3, not a new port), and the direction of SMTP 25 vs 587 (25 = server↔server, 587 = client→server).
