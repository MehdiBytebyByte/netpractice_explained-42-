<div align="center">

# NetPractice

**A practical introduction to networking & subnetting** · _42 Network Project_

Ten simulator levels where you configure **IP addresses**, **subnet masks** and **routing tables** so that every pair of machines can talk to each other.

</div>

---

## Table of Contents

- [What is NetPractice?](#what-is-netpractice)
- [What is in this repository?](#what-is-in-this-repository)
- [How the simulator works](#how-the-simulator-works)
- [Networking basics](#networking-basics)
  - [The layers of networking](#the-layers-of-networking)
  - [The TCP/IP model](#the-tcpip-model)
  - [Encapsulation](#encapsulation)
- [IPv4 addressing](#ipv4-addressing)
  - [Anatomy of an IPv4 address](#anatomy-of-an-ipv4-address)
  - [Network part vs host part](#network-part-vs-host-part)
  - [The subnet mask](#the-subnet-mask)
  - [CIDR notation](#cidr-notation)
- [Subnetting, step by step](#subnetting-step-by-step)
  - [Why subnet?](#why-subnet)
  - [Finding the network address](#finding-the-network-address)
  - [Usable hosts](#usable-hosts)
  - [The three special addresses](#the-three-special-addresses)
  - [Reference: common masks](#reference-common-masks)
- [Routers, gateways & routing](#routers-gateways--routing)
  - [Why packets need routing](#why-packets-need-routing)
  - [Routing table fields](#routing-table-fields)
  - [Same subnet vs different subnet](#same-subnet-vs-different-subnet)
  - [The default gateway](#the-default-gateway)
- [How the answer files map to levels](#how-the-answer-files-map-to-levels)
- [A worked example](#a-worked-example)
- [Tips for the simulator](#tips-for-the-simulator)
- [Resources](#resources)

---

## What is NetPractice?

**NetPractice** is the 42 school's introduction to **computer networking**. It is not a coding project — it is a **web-based simulator** made of **10 levels**. In every level you are shown a small network diagram (machines, routers, switches, links) and you must **fill in the missing configuration** so that every machine can reach every other machine.

You configure two kinds of things:

1. **Interfaces** — the IP address and subnet mask of a network card (on a host or on a router).
2. **Routing tables** — the rules a router uses to know where to send packets (which network via which neighbor/gateway).

The simulator checks your answers (you can even get **3 tries per level** on some checks) and only lets you proceed to the next level once the whole network works.

> The evaluation is an oral: your evaluator opens random levels and you must **explain** your choices and complete them live. That is why it is worth genuinely understanding the concepts below instead of just memorizing answers.

---

## What is in this repository?

This repository contains **only the completed answers** — one JSON file per level:

```text
netpractice_answers/
├── level1.json
├── level2.json
├── ...
└── level10.json
```

Each file stores the values you would type into the simulator:

| JSON key | Meaning                                                                                  |
| -------- | ---------------------------------------------------------------------------------------- |
| `ifs`    | **Interfaces** — per network card, the `ip` and/or `mask` to set                         |
| `routes` | **Routing tables** — per device, the `route` (destination network) and `gate` (next hop) |

Example — `level1.json`:

```json
{
  "routes": {},
  "ifs": {
    "A1": { "ip": "104.98.23.11" },
    "B1": {},
    "C1": {},
    "D1": { "ip": "211.191.249.74" }
  }
}
```

These files are a **cheat-sheet of the correct configurations**. To actually see the diagrams and run the levels you need the official simulator (see [Resources](#resources)).

---

## How the simulator works

A level shows a set of **devices** connected by links:

- **Hosts / computers** — have one network interface each (e.g. `A1`, `B1`, `C1`, …).
- **Routers** — have **several** interfaces, one per subnet they connect to (e.g. `R1`, `R2`, …). They move packets between subnets.
- **Switches / "the Net"** — just pass traffic around on the same subnet; they do **not** need an IP.

The golden rules of every level are always the same:

1. Two devices **directly connected** on a link must be in the **same subnet**.
2. **Every interface** on a device must be in a different subnet from the others (unless they are in different broadcast domains).
3. A router needs a **routing table entry** for any subnet that is **not directly attached** to it.
4. If the destination is not on the local subnet, the sender must send the packet to its **default gateway**.

---

## Networking basics

### The layers of networking

Networking is usually described as a **stack of layers**. Each layer has one job and hands its work to the next. The classic reference model is the **OSI model** (7 layers). The real-world model used on the Internet is the **TCP/IP model** (4 layers) — which is the one that matters for NetPractice.

![OSI model](https://commons.wikimedia.org/wiki/Special:FilePath/OSI_Model_v1.svg)

_The 7-layer OSI model. TCP/IP merges these into 4 layers (see below)._

| Layer (OSI) | Name         | Job (in one line)                                      | Examples              |
| ----------- | ------------ | ------------------------------------------------------ | --------------------- |
| 7           | Application  | Services the user sees                                 | HTTP, HTTPS, DNS, SSH |
| 6           | Presentation | Encoding / encryption of data                          | TLS, SSL              |
| 5           | Session      | Managing a conversation between two apps               | NetBIOS               |
| 4           | Transport    | End-to-end delivery between **applications** (ports)   | TCP, UDP              |
| 3           | Network      | Delivery between **hosts**, addressing & routing       | IP, ICMP              |
| 2           | Data Link    | Delivery on the **local link** (MAC addresses, frames) | Ethernet, Wi-Fi       |
| 1           | Physical     | Raw bits on the wire                                   | Cables, radio         |

### The TCP/IP model

The Internet actually runs on a simpler, **4-layer** model. NetPractice deals almost entirely with the **Network layer (IP)** and touches the **Link layer** implicitly.

| TCP/IP layer           | What lives here                                 | NetPractice relevance            |
| ---------------------- | ----------------------------------------------- | -------------------------------- |
| **Application**        | HTTP, SSH, DNS                                  | Just the "thing" that sends data |
| **Transport**          | TCP, UDP (ports)                                | Not really configured            |
| **Internet / Network** | **IP addresses**, **subnet masks**, **routing** | **Everything you configure**     |
| **Link**               | Ethernet, MAC addresses                         | The links between boxes          |

![Basic TCP/IP node](https://commons.wikimedia.org/wiki/Special:FilePath/Basic%20TCP-IP%20node-en.svg)

_A device speaking TCP/IP: applications on top, IP in the middle, the physical link below._

The two protocols you will hear about constantly — **TCP** and **IP** — live on different layers:

- **IP** (Internet Protocol, layer 3) does the _addressing_: it gives every machine a unique address and routes packets from network to network. It is **connectionless** — packets find their own way, hop by hop.
- **TCP** (Transmission Control Protocol, layer 4) does the _reliable delivery_: it sits on top of IP, splits data into segments, numbers them, re-orders them, and re-sends anything lost.

> For NetPractice you only reason about **IP**: same subnet → send directly; different subnet → send via a router. TCP is already handled for you.

### Encapsulation

When a message travels down the stack, each layer **wraps** the previous data with its own header. This is called **encapsulation**.

![Encapsulation in TCP-IP](https://commons.wikimedia.org/wiki/Special:FilePath/TCP-IP%20zapouzdreni%20-%20edited.svg)

_Application data is wrapped in a TCP header, then an IP header, then an Ethernet header before going on the wire._

The receiving machine does the reverse (**decapsulation**): each layer strips its header and passes the payload up. This is why the same IP packet can carry a web request, an email, or a game update — the IP layer does not care what the transport layer put inside.

---

## IPv4 addressing

### Anatomy of an IPv4 address

An IPv4 address is a **32-bit number**, written for humans as four **octets** (8-bit groups) in decimal, separated by dots:

```
172.16.254.1   =   10101100 . 00010000 . 11111110 . 00000001
                   └──172──┘  └──16───┘  └──254──┘  └──1────┘
```

Each octet goes from `0` to `255`, so the full range is `0.0.0.0` → `255.255.255.255`.

![IPv4 address structure](https://commons.wikimedia.org/wiki/Special:FilePath/IPv4%20address%20structure%20and%20writing%20systems-en.svg)

_Every IPv4 address is 32 bits split into 4 octets, and is split again into a network part and a host part._

### Network part vs host part

This is **the** key idea of the whole project:

- The **network part** (prefix) identifies _which subnet_ the machine belongs to. All machines in the same subnet share the same prefix.
- The **host part** identifies _which machine_ within that subnet.

The **subnet mask** tells you exactly where the boundary between the two parts lies.

### The subnet mask

The subnet mask is also a 32-bit number, but it has a special shape: a run of **1-bits** (the network part) followed by a run of **0-bits** (the host part). The `1`s are said to "cover" (mask) the network bits.

For example the mask `255.255.255.0` in binary is:

```
255 . 255 . 255 . 0
11111111 . 11111111 . 11111111 . 00000000
└── 24 network bits ──┘ └─ 8 host bits ─┘
```

To find the **network address** of any IP, you do a **bitwise AND** between the IP and the mask. The mask "keeps" the network bits and "zeroes out" the host bits:

```
IP     172.16.254.1   10101100 . 00010000 . 11111110 . 00000001
MASK   255.255.255.0  11111111 . 11111111 . 11111111 . 00000000
AND   ----------------------------------------------
NET    172.16.254.0   10101100 . 00010000 . 11111110 . 00000000
```

So `172.16.254.1/24` belongs to the network `172.16.254.0`.

### CIDR notation

Writing the full mask is verbose, so we use **CIDR notation**: a slash followed by the **number of network bits**.

| Mask              | CIDR  | Host bits | Usable hosts |
| ----------------- | ----- | --------- | ------------ |
| `255.255.255.0`   | `/24` | 8         | 254          |
| `255.255.255.128` | `/25` | 7         | 126          |
| `255.255.255.192` | `/26` | 6         | 62           |
| `255.255.255.252` | `/30` | 2         | 2            |

In the answer files you will sometimes see **both** forms, e.g. `"mask": "255.255.255.128"` on an interface but `"route": "168.129.218.2/16"` in a routing table.

---

## Subnetting, step by step

### Why subnet?

Subnetting = **borrowing host bits to create smaller networks**. Instead of one giant flat network, you split it into several smaller ones:

- traffic is contained (less congestion, more security),
- routing is cleaner,
- you do not waste addresses.

### Finding the network address

Given an IP and its mask:

1. Write the mask in binary.
2. Count the `1`s → that is your prefix length (`/n`).
3. Zero out all host bits of the IP (or AND with the mask) → you get the **network address**.

Example — `192.168.1.37` with mask `255.255.255.224` (`/27`):

```
192.168.1.37    = 11000000.10101000.00000001.00100101
255.255.255.224 = 11111111.11111111.11111111.11100000
NETWORK         = 11000000.10101000.00000001.00100000 = 192.168.1.32
```

The host part has 5 bits, so the block spans host bits `00000` → `11111`, i.e. `192.168.1.32` → `192.168.1.63`.

### Usable hosts

If the host part has **h** bits, the subnet contains **2^h** addresses, but **2 are not usable by hosts**:

- the all-`0` host address = the **network address**,
- the all-`1` host address = the **broadcast address**.

So the number of usable host addresses is:

```
usable hosts = 2^h − 2
```

| CIDR  | h (host bits) | Total addresses | Usable hosts                |
| ----- | ------------- | --------------- | --------------------------- |
| `/24` | 8             | 256             | 254                         |
| `/25` | 7             | 128             | 126                         |
| `/26` | 6             | 64              | 62                          |
| `/27` | 5             | 32              | 30                          |
| `/28` | 4             | 16              | 14                          |
| `/29` | 3             | 8               | 6                           |
| `/30` | 2             | 4               | **2**                       |
| `/31` | 1             | 2               | 2 (special, point-to-point) |
| `/32` | 0             | 1               | 1 (a single host)           |

> NetPractice loves **`/30`** links between two routers: only 2 usable addresses — exactly one for each end of the link.

### The three special addresses

Every subnet has three reserved addresses you must never assign to a normal host:

| Address       | Value                             | Purpose                                                          |
| ------------- | --------------------------------- | ---------------------------------------------------------------- |
| **Network**   | host bits all `0`                 | Identifies the subnet itself (used in routing tables)            |
| **Broadcast** | host bits all `1`                 | Sends a packet to _every_ host on the subnet                     |
| **Gateway**   | usually first or last usable host | The router's IP on this subnet (what hosts send to when leaving) |

For `192.168.1.32/27`:

- network = `192.168.1.32`
- first usable host = `192.168.1.33`
- last usable host = `192.168.1.62`
- broadcast = `192.168.1.63`

### Reference: common masks

| CIDR  | Mask              | Networks in a class-C (`/24`) | Usable hosts per subnet |
| ----- | ----------------- | ----------------------------- | ----------------------- |
| `/24` | `255.255.255.0`   | 1                             | 254                     |
| `/25` | `255.255.255.128` | 2                             | 126                     |
| `/26` | `255.255.255.192` | 4                             | 62                      |
| `/27` | `255.255.255.224` | 8                             | 30                      |
| `/28` | `255.255.255.240` | 16                            | 14                      |
| `/29` | `255.255.255.248` | 32                            | 6                       |
| `/30` | `255.255.255.252` | 64                            | 2                       |

![Subnetting concept](https://commons.wikimedia.org/wiki/Special:FilePath/Subnetting_Concept-en.svg)

_Subnetting a `200.100.10.0/24` network into two `/25` halves: each gets its own network and broadcast address._

---

## Routers, gateways & routing

### Why packets need routing

On a **local subnet**, machines talk directly (they are on the same link). But a machine can only talk directly to machines **in its own subnet**. To reach anything else it must hand the packet to a **router**, which forwards it hop by hop toward the destination.

A router is a machine with **several interfaces**, each connected to a different subnet — its job is to decide, for every incoming packet, **which interface to send it out of**.

### Routing table fields

The answer files store routing entries as `{ "route": …, "gate": … }`. These map to the classic routing-table columns:

| Field                  | Meaning                                                                                                |
| ---------------------- | ------------------------------------------------------------------------------------------------------ |
| **Destination**        | The target network (e.g. `10.0.0.0/24`) or `default` / `0.0.0.0/0`                                     |
| **Gateway / Next hop** | The IP of the _next_ router to hand the packet to (must be reachable — on a directly connected subnet) |
| **Interface**          | Which of the router's own interfaces to send it out of (usually implied by the gateway)                |

### Same subnet vs different subnet

This decision is made on **every** device, every time it sends a packet:

```
Is destination in my subnet?  (AND dest with my mask == my network)
    ├─ YES  → send DIRECTLY to that host on the local link (ARP to find its MAC)
    └─ NO   → send the packet to my DEFAULT GATEWAY
```

### The default gateway

The **default route** catches everything that does not match a more specific route. It is written as `default`, `0.0.0.0/0`, or sometimes just given by a gateway IP.

> **Rule of thumb for NetPractice:** every **host** needs a default gateway pointing at the router on its subnet. Every **router** needs:
>
> - direct routes for the subnets physically attached to it (usually automatic — the interface itself),
> - a default route (or specific routes) for anything further away,
> - and the _next_ router must have a route **back** to the source's network (routing is two-way!).

---

## How the answer files map to levels

Reading the JSONs, you can reconstruct what each level is teaching:

| Level | Main concept being tested                                                                   |
| ----- | ------------------------------------------------------------------------------------------- |
| 1     | Two hosts on the same subnet — same mask, different host part                               |
| 2     | Different subnets joined by a router — each side gets its own mask                          |
| 3     | Two subnets on opposite sides of a router; choose masks so both sides are consistent        |
| 4     | Router between two subnets; assign the router an IP in each subnet                          |
| 5     | **Routing tables** appear: hosts need a default route via their gateway                     |
| 6     | Internet access — routing to "Somewhere on the Net", `0.0.0.0/0`                            |
| 7     | Two routers in series; every device needs a route to the far network                        |
| 8     | More complex inter-router routing with `/30` point-to-point links                           |
| 9     | **CIDR** in routes (`/18`, `/25`, `/30`) + multi-router path selection                      |
| 10    | The full picture: several routers, several subnets, correct masks **and** routes everywhere |

---

## A worked example

Let's look at what `level1.json` tells us and why it is correct:

```json
{
  "routes": {},
  "ifs": {
    "A1": { "ip": "104.98.23.11" },
    "B1": {},
    "C1": {},
    "D1": { "ip": "211.191.249.74" }
  }
}
```

Level 1 is the simplest diagram — two pairs of machines, and for each pair **only one IP is given**; you must type the **other one** so that the two machines are on the **same subnet**.

`A1` has `104.98.23.11`. To put a machine on the _same_ network, its IP must match `A1` for all the bits covered by the mask (here a plain classful `/24`, i.e. the first three octets): a correct answer is any `104.98.23.x` (with `x` not already used). That is why the repository answer for `B1` leaves `ip` blank in the file — in the simulator you fill it — while `C1`/`D1` form the second independent pair (`211.191.249.74` and a matching `211.191.249.y`).

> The key insight: **it does not matter what the "right" IP is in the abstract — it only matters that the two ends of a link agree on the same subnet** (same network bits), with unique host bits, and that routers/gateways are consistent.

---

## Tips for the simulator

1. **Start from the links.** Two boxes joined by a cable must be in the same subnet. Pick a mask, then give each side an IP with the same network part and a different host part.
2. **Routers have many faces.** Every interface of a router belongs to a _different_ subnet. Give each interface an IP that matches the subnet of the devices on that side.
3. **Respect `/30` point-to-point links.** When two routers are directly connected, `/30` gives you exactly 2 usable addresses — use one for each end.
4. **Do not use network or broadcast addresses** as a machine/gateway IP.
5. **When in doubt about a host: default gateway.** A host that needs to reach another subnet must have a default route via its local router.
6. **Routing is round-trip.** If A can reach B, B must also be able to reach A. Check the return path.
7. **`0.0.0.0/0` = default.** A route to `0.0.0.0/0` means "anything not matched elsewhere goes here" — usually toward the Internet.
8. **Use the built-in validation.** The simulator tells you (with a limited number of attempts) whether each interface / route is valid. Iterate.

---

## Resources

- **42 subject** — available on your intra page for the project (search "NetPractice").
- **Official simulator** — launched from the 42 intra project page (web app).
- [Subnetting — Wikipedia](https://en.wikipedia.org/wiki/Subnetwork)
- [IPv4 subnetting reference — Wikipedia](https://en.wikipedia.org/wiki/IPv4_subnetting_reference)
- [What is a subnet? — Cloudflare Learning](https://www.cloudflare.com/learning/network-layer/what-is-a-subnet/)
- [OSI model — Cloudflare Learning](https://www.cloudflare.com/learning/ddos/glossary/open-systems-interconnection-model-osi/)
- [What is TCP/IP? — Cloudflare Learning](https://www.cloudflare.com/learning/network-layer/what-is-tcp-ip/)
- [Subnet calculator — CIDR.xyz](https://cidr.xyz/)

---

<div align="center">

_Self-contained networking notes for the 42 **NetPractice** project — built from the `level_.json` answer files in this repository.\*

</div>
