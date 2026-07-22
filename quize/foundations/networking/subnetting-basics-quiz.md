# Subnetting Basics — Quiz

Questions I got asked on this topic, written down so I can re-test myself later instead of just re-reading the note and assuming I remember it.

## How to use this

Don't scroll straight down. Read a question, answer it out loud like I'm in an interview, then open the answer and check. If I got it half right, that still counts as wrong — the point of this file is catching the gap between "I know the shape of it" and "I can say it precisely."

**The one picture to hold in your head for all of this:** an IP address is just 32 light switches in a row, each one ON (1) or OFF (0). The subnet mask is a second row of 32 switches that tells you which of the address's switches are "locked" (shared by everyone on the network — like a building name) and which are "free" (unique to one device — like an apartment number). Every question below is just some version of "count the locked switches" or "do something with the free ones."

---

## Round 1 — Scenarios

**1. You're handed a subnet mask of 255.192.0.0 and asked for the CIDR notation. What is it, and what's the trap here if you try to shortcut it?**

<details>
<summary>Answer</summary>

**Easy way to think about it:** forget the decimal numbers for a second — they're disguises. Turn each of the 4 numbers into 8 light switches (bits), then count how many are ON from the left. That count *is* the CIDR number.

```
255 = 11111111  → all 8 ON
192 = 11000000  → only 2 ON
0   = 00000000  → 0 ON
0   = 00000000  → 0 ON
```
Total switches ON = 8 + 2 + 0 + 0 = **/10**

**The trap:** the number "192" also shows up in `255.255.255.192` (a /26). If you just recognize "192" on autopilot, your brain jumps to "/26" because that's the version you've seen more. But here 192 sits in the *2nd* box, not the 4th — same number, totally different meaning. *Where* the switches are matters as much as how many there are. Always convert and count; never pattern-match the number alone.

</details>

**2. A ticket says a subnet has a /26 mask. Someone tells you "that gives you 64 usable hosts." Are they right?**

<details>
<summary>Answer</summary>

**Easy way to think about it:** a subnet is like a small street of houses. The total number of *addresses* on the street isn't the same as the number of houses you can actually move into — one address is always "the street sign" (network ID) and one is always "the shared mailbox" (broadcast). Neither is a real house.

64 is the total addresses on the street. It is **not** the number of move-in-ready houses.

```
/26 → 26 switches locked, 32 − 26 = 6 free switches
6 free switches → 2×2×2×2×2×2 = 64 total addresses
```

Now remove the 2 that are never real houses:
```
64 − 2 = 62 usable hosts
```

The person is wrong — it's **62**, not 64. Forgetting the −2 is the #1 way people get the right method but the wrong final number. Whenever a question says "usable hosts," the −2 step happens automatically, every time.

</details>

**3. You're splitting a /24 network into /28 subnets. How many subnets do you get, and how many usable hosts per subnet?**

<details>
<summary>Answer</summary>

**Easy way to think about it:** you have one big pizza (/24 = 256 slices total) and you're cutting it into smaller pizzas (/28 subnets). Two questions: how many small pizzas do you get, and how many slices can you actually eat from each one (minus the 2 slices that are always "display only" — network ID and broadcast)?

**How many small pizzas (subnets)?**
Going from /24 to /28 moves 4 extra switches from "free" to "locked" (28 − 24 = 4). Each newly locked switch doubles how many pieces you can cut into:
```
2×2×2×2 = 16 subnets
```

**How many usable slices per small pizza?**
```
Free switches left = 32 − 28 = 4
2×2×2×2 = 16 total addresses per subnet
16 − 2 = 14 usable hosts
```

**Sanity check with plain numbers:** big pizza = 256 slices, small pizza = 16 slices, 256 ÷ 16 = 16 pizzas. Same answer, easy to double-check.

</details>

**4. You're given IP `10.20.5.200` with mask `255.255.255.240`. What's the network ID? Someone on your team just writes down `10.20.5.200` as the network ID — what did they get wrong?**

<details>
<summary>Answer</summary>

**Easy way to think about it:** the network ID is what you get by turning off every switch that isn't nailed down. Keep the locked switches exactly as they are, force every free switch to OFF (0), no matter what it currently shows. Your teammate just wrote the original address again — that skips the whole "turn off the free switches" step.

```
Mask 255.255.255.240 → last box: 11110000  (first 4 locked, last 4 free)
IP's last box, 200 →              11001000

Keep the locked 4, force the free 4 to 0:
                                    1100 0000  =  192
```

**Network ID: `10.20.5.192`**

Think of it as a street called "10.20.5.192" — the actual houses run `.193` to `.206` (the usable range), `.192` is the street sign, `.207` is the shared mailbox. `.200` is a real house on this street — it's just not the street sign. The mistake is calling the house's own address the street sign, instead of doing the AND to find the real one.

</details>

**5. Two devices are plugged into the same physical switch, both configured with mask 255.255.255.192:**
**Device A: 192.168.10.65**
**Device B: 192.168.10.130**
**Can they talk directly, or does traffic need to go through a router? Show the network ID for each.**

<details>
<summary>Answer</summary>

**Easy way to think about it:** two houses can plug into the same power strip (the switch) and still be on completely different streets. Being on the same hardware says nothing about whether they're logical neighbors. The only way to know is to find each house's street name (network ID) and see if the names match.

Mask 255.255.255.192 → last box pattern: `11000000` (2 locked, 6 free)

```
Device A: 65  = 01000001
          AND  11000000
          =    01000000  = 64   → street name: 192.168.10.64

Device B: 130 = 10000010
          AND  11000000
          =    10000000  = 128  → street name: 192.168.10.128
```

**Different street names → different subnets.** Even though A and B share a switch and the exact same mask, they live on two different logical streets (`.64` and `.128`), and a plain switch can't move traffic between two streets. You need a **router**.

The trap: seeing "same mask" and stopping there, treating it as proof. Same mask just means "same size street." You still have to check the actual street name for each house.

</details>

**6. Mask `255.255.255.224`. Give the CIDR, usable hosts, and how many subnets you'd get carving up a /24 — all three in one answer.**

<details>
<summary>Answer</summary>

**Easy way to think about it:** one mask, three questions — but all three come from the same pair of numbers: how many switches are locked, and how many are free. Find that split first, then everything else is quick arithmetic.

```
255.255.255.224 → last box: 224 = 11100000  (3 locked, 5 free)
```

**CIDR:** 8 + 8 + 8 + 3 locked = **/27**

**Usable hosts** (uses the 5 free switches):
```
2×2×2×2×2 = 32 total addresses
32 − 2 = 30 usable hosts
```

**Subnets from a /24** (uses "how many switches did I newly lock" = 27 − 24 = 3 borrowed):
```
2×2×2 = 8 subnets
```

Same starting point (locked vs. free switches), three different questions. Once you have that split, nothing else needs memorizing.

</details>

**7. You're doing a design review and someone claims two hosts "must be on the same subnet, they've both got a /24 mask." Is a matching prefix length by itself enough to conclude that?**

<details>
<summary>Answer</summary>

**Easy way to think about it:** two people both saying "I live in a 5-digit zip code" doesn't mean they live in the *same* zip code — it just means their zip codes are the same length. `192.168.1.50` and `192.168.2.50` are both /24, but /24 just says "the first 3 boxes are locked, the last box is free" for *each of them individually*. It doesn't say the first 3 boxes have the *same numbers* between the two.

```
192.168.1.50  → locked part = 192.168.1
192.168.2.50  → locked part = 192.168.2
```

Different locked parts → different streets → **not** the same subnet, despite the identical mask.

The only real test: apply the mask to each IP, get each network ID, and literally compare the two. Match → same subnet. No match → you need a router, no matter how similar the addresses or masks look at a glance.

</details>

**8. You see an internal address `172.20.15.4` and someone flags it as "not a valid private IP" because they remember the private range as `172.16.x.x`–`172.31.x.x` and `20` "isn't 16." Are they right to flag it?**

<details>
<summary>Answer</summary>

**Easy way to think about it:** they memorized the *answer* (16 through 31) without knowing *why* it's the answer — like memorizing a times-table result without understanding multiplication. The moment a number in the middle of the list shows up, it feels unfamiliar even though it's totally valid.

`172.16.0.0/12` really means: the whole first box (`172`) is locked, **plus** the first 4 switches of the second box are locked — leaving 4 free switches in that second box.

```
4 free switches → 2×2×2×2 = 16 possible values
```

Starting at 16 (00010000) and counting up 16 values lands at 31 (00011111) — so **every** number from 16 to 31 is valid, including 20, 24, 29, all of them, since they're just different combinations of those same 4 free switches.

**So no — the flag is wrong.** `172.20.15.4` is a perfectly normal private address. The real fix isn't "memorize 16–31 harder" — it's "know it's 4 free bits, so of course it's a run of 16 consecutive numbers."

</details>

---

## Round 2 — Quick-fire recall

Same rules — say the answer out loud before opening it. No context given on purpose; this round tests raw recall under pressure, since that's where the gaps actually show up.

<details>
<summary>1. What does the "/N" in CIDR notation actually count?</summary>The number of locked (1) switches/bits in the subnet mask, counted from the left</details>
<details>
<summary>2. What's the formula for total addresses in a subnet?</summary>2^(free bits) — e.g. 6 free bits = 2×2×2×2×2×2 = 64 total addresses</details>
<details>
<summary>3. What's the formula for usable hosts?</summary>2^(free bits) − 2 — total addresses, minus the 2 that are never real devices</details>
<details>
<summary>4. Why subtract 2 for usable hosts?</summary>One address is always the "street sign" (network ID), one is always the "shared mailbox" (broadcast) — neither can be a device</details>
<details>
<summary>5. How do you compute a network ID from an IP and mask?</summary>Line up the IP's switches with the mask's switches. Keep the locked ones as-is, force every free one to 0</details>
<details>
<summary>6. How do you compute a broadcast address from an IP and mask?</summary>Same idea, but force every free switch to 1 instead of 0</details>
<details>
<summary>7. What's the formula for subnets gained by borrowing bits?</summary>2^(borrowed bits) — borrowed bits = new prefix minus old prefix (how many switches you newly locked)</details>
<details>
<summary>8. Does having the same subnet mask mean two devices are on the same subnet?</summary>No — same mask just means "same size street." You still have to compute and compare their actual network IDs (street names)</details>
<details>
<summary>9. /24 in binary — how many switches locked, and what's the decimal mask?</summary>24 locked switches, 255.255.255.0</details>
<details>
<summary>10. /16 in decimal?</summary>255.255.0.0</details>
<details>
<summary>11. /26 in decimal?</summary>255.255.255.192</details>
<details>
<summary>12. /27 in decimal?</summary>255.255.255.224</details>
<details>
<summary>13. /28 in decimal?</summary>255.255.255.240</details>
<details>
<summary>14. How many usable hosts does a /30 give, and what is a /30 commonly used for?</summary>2 usable hosts (2×2 − 2 = 2) — classic use is a tiny link with just 2 devices, like two routers wired directly to each other</details>
<details>
<summary>15. What's the private IP range 172.16.0.0/12, expressed as the second-box span?</summary>172.16.x.x through 172.31.x.x — a run of 16 numbers, because that octet has exactly 4 free switches (2×2×2×2 = 16)</details>
<details>
<summary>16. If you go from a /24 to a /26, how many switches did you lock, and how many streets (subnets) does that create?</summary>2 newly locked switches, 2×2 = 4 subnets</details>
<details>
<summary>17. What is the network ID address used for — can it be assigned to a device?</summary>It's the "street sign" that names the subnet itself — it can never be handed out to an actual device</details>
<details>
<summary>18. What is the broadcast address used for — can it be assigned to a device?</summary>It means "send this to every device on the street at once" — also never handed out to an actual device</details>

**Known trip-up points — drill these specifically:**
- Total addresses and usable hosts are **not the same number**. Total addresses is 2^(free bits). Usable hosts is that number minus 2. Stopping at the first one and calling it "usable hosts" is the single most repeated mistake — think "street full of addresses" vs. "actual houses you can move into."
- To find a network ID or broadcast address, don't just re-write the original IP. Apply the mask: locked switches stay, free switches get forced to 0 (network ID) or 1 (broadcast).
- Same subnet mask does **not** mean same subnet — it just means the two "streets" are the same length. Always compute and compare the actual network ID before assuming two devices can talk directly.
- Convert masks to binary and count the switches — don't recognize a number like "192" and assume it means the same thing it meant somewhere else. Where the number sits matters as much as what the number is.
- Private ranges like 172.16.0.0/12 aren't a list to memorize — they come from counting free bits. Knowing *why* the range is 16 numbers wide (2^4) means a number in the middle of that range never throws you off.
- Say "leftmost N bits/switches," not "leftmost N digits." Thinking in digits instead of bits is the root cause of most mask-counting mistakes.
