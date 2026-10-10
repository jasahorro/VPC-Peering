# VPC Peering — Cheat Sheet

Lesson notes for reviewing later. The project write-up is in [README.md](README.md).

## Contents

1. [What I built](#what-i-built)
2. [Key terms](#key-terms)
3. [Setting up the VPCs](#setting-up-the-vpcs)
4. [Peering connections](#peering-connections)
5. [Route tables](#route-tables)
6. [Launching the EC2 instances](#launching-the-ec2-instances)
7. [Troubleshooting EC2 Instance Connect](#troubleshooting-ec2-instance-connect)
8. [Elastic IPs](#elastic-ips)
9. [Ping and ICMP](#ping-and-icmp)
10. [Build order and cleanup](#build-order-and-cleanup)
11. [Self-quiz](#self-quiz)

---

## What I built

- 🌉 **Two VPCs joined by a peering connection**, so they can talk to each other using private IP addresses without going over the public internet.
- 🚏 **Route table entries** in both VPCs that send traffic for the other VPC through the peering connection.
- 💻 **An EC2 instance in each VPC**, used to test the connection with `ping`.

```
        NextWork-1-vpc (10.1.0.0/16)                  NextWork-2-vpc (10.2.0.0/16)
  ┌──────────────────────────────────┐         ┌──────────────────────────────────┐
  │  Instance - NextWork VPC 1       │  ping   │  Instance - NextWork VPC 2       │
  │  (Elastic IP for Instance        │ ──────► │  SG allows All ICMP - IPv4       │
  │   Connect; SG allows SSH 22)     │  (ICMP) │  from 10.1.0.0/16                │
  └──────────────────────────────────┘         └──────────────────────────────────┘
   Route: 10.2.0.0/16 → pcx-…   ◄──── peering connection (pcx-…) ────►   Route: 10.1.0.0/16 → pcx-…
          (Requester)                                                         (Accepter)
```

## Key terms

| Term | Plain meaning | Analogy |
|---|---|---|
| **VPC** | Your own private, isolated network inside AWS | A gated neighbourhood |
| **CIDR block** | The range of private IP addresses a VPC owns (e.g. `10.1.0.0/16`) | The neighbourhood's range of street numbers |
| **VPC peering connection** | A direct private link between two VPCs | A private bridge between two gated neighbourhoods |
| **Requester / Accepter** | The VPC that sends the peering invite / the VPC that approves it | Sending a friend request / accepting it |
| **Route table** | Rules that say where traffic for each destination should go | Road signs at the gate |
| **Tenancy** | Whether your instances share hardware with other customers | Renting a flat in a block vs. a whole house |
| **NAT gateway** | Lets private subnets reach out to the internet, but blocks inbound | A one-way door: you can go out, strangers can't come in |
| **Internet gateway** | Lets public subnets talk to the internet both ways | A two-way front door |
| **VPC endpoint** | Private connection from your VPC to AWS services like S3 | A staff-only corridor to the shop, instead of walking around the street |
| **EC2 Instance Connect** | Browser-based SSH; AWS pushes a temporary key for you | A visitor pass printed at reception, so you don't carry your own key |
| **Elastic IP** | A static public IPv4 address you own until you release it | A permanent home address instead of moving every restart |
| **Ping / ICMP** | A tool that checks if another machine responds; ICMP is the protocol it uses | Ringing someone's doorbell to see if they're home |
| **Network interface** | The virtual network card linking an instance to the VPC | The network cable plugged into a computer |

## Setting up the VPCs

- Used the **VPC and more** wizard to create two VPCs, each with **1 public subnet and 0 private subnets**.
- CIDR blocks: **`10.1.0.0/16`** for VPC 1 and **`10.2.0.0/16`** for VPC 2. Numbering them to match the VPC names makes them easy to remember.
- **The CIDR blocks must not overlap.** If they did, a route table couldn't tell whether traffic should stay local or go through the peering connection.

**Tenancy:**

| Option | What it means | When to use |
|---|---|---|
| **Default** — *used here* | Shares hardware with other AWS customers | Most workloads; cheaper |
| **Dedicated** | Hardware used only by you | Strict compliance or isolation, e.g. a healthcare company with patient data. Costs more |

**NAT gateway vs internet gateway:**

| | Internet gateway | NAT gateway |
|---|---|---|
| Used by | Public subnets | Private subnets |
| Outbound to internet | ✅ | ✅ (e.g. downloading security updates) |
| Inbound from internet | ✅ | ❌ blocked |

**VPC endpoints** keep traffic to AWS services (like S3) inside the AWS network instead of the public internet. That's more secure and can cut data transfer costs.

## Peering connections

- A **peering connection** lets two VPCs send traffic to each other using **private IP addresses**.
- Without it, resources in different VPCs would have to use **public** addresses and talk over the public internet.
- **Requester** = the VPC that sends the invite (NextWork-1-vpc). **Accepter** = the VPC that receives it and must accept (NextWork-2-vpc).
- The connection doesn't work until the Accepter **accepts** it.

> **Extra:** peering is **not transitive**. If A is peered with B, and B with C, A still can't reach C. A and C need their own peering connection.

## Route tables

- Accepting the peering connection isn't enough. Each VPC still needs a **route** telling traffic for the other VPC to use the peering connection.

| VPC | Destination | Target |
|---|---|---|
| NextWork-1-vpc | `10.2.0.0/16` | Peering connection (`pcx-…`) |
| NextWork-2-vpc | `10.1.0.0/16` | Peering connection (`pcx-…`) |

- **Lesson's deliberate typo:** entering `10.1.0.0/16` as the destination in VPC 1's route table gives an error, because it's VPC 1's *own* CIDR block. Route tables can't tell local and remote destinations apart if they share a CIDR block. That's why the CIDRs must be unique.

## Launching the EC2 instances

- **Network settings must be edited**: by default instances launch into the account's **default VPC**, so you have to pick NextWork-1-vpc (or 2) yourself.
- **No key pair** this time: EC2 Instance Connect manages a temporary key for you. Key pairs were already practised in earlier projects.
- **Auto-assign public IP** was left on **Disable**. That caused the next problem.

## Troubleshooting EC2 Instance Connect

Two problems had to be fixed before connecting to Instance 1:

| Problem | Cause | Fix |
|---|---|---|
| "No public IPv4 address assigned" | Auto-assign public IP was disabled at launch | Allocate an **Elastic IP** and associate it with the instance |
| Connection still blocked | The default security group only allows inbound traffic from inside the VPC, and Instance Connect comes from the internet | Add an inbound rule: **SSH, port 22** |

> **Key takeaway:** to use EC2 Instance Connect, the instance needs a **public IP** and must be in a **public subnet**, because Instance Connect connects over the internet.

## Elastic IPs

- A **static** public IPv4 address allocated to your account. You choose which instance it's attached to.
- Normal public IPs are **dynamic**: they change when the instance is stopped and started.
- **Why companies use them:** if a server's IP changes, the website's DNS record must be updated, and DNS changes can take hours or days to spread. With an Elastic IP the address never changes, so no DNS update is needed.
- **Why we used one:** it's a way to give an instance a public IP **after** it has already launched.
- **Public IPv4 address pool:** where the address comes from. Default is **Amazon's pool**. You can also **Bring Your Own IP (BYOIP)**.
- **Private IP address field:** only matters if the instance has several private IPs. An instance *can* have several, which lets one server host multiple apps with separate network identities.
- **Releasing** an Elastic IP returns it to Amazon's pool, where it may later go to another AWS user.

## Ping and ICMP

- **Ping** sends a small packet to an IP address and waits for a reply ("hello, are you there?").
- Run from Instance 1: `ping <private IPv4 of Instance 2>` (e.g. `ping 10.2.0.x`). Use the instance's IP, not the VPC ID or the CIDR's network address.

| What you see | Meaning |
|---|---|
| One line, then nothing | Ping was sent but no replies came back. Something is blocking it |
| A new line every second | Success: each line is a reply from Instance 2 |

- **Why it failed at first:** Instance 2's security group blocked **ICMP**, the protocol ping uses. Blocking ICMP by default helps stop attacks that flood a server with pings.
- **Fix:** on Instance 2's security group, add inbound rule **All ICMP - IPv4** with source **`10.1.0.0/16`** (VPC 1's CIDR).
- **All ICMP - IPv4** allows every kind of ICMP message, not just ping. There's a separate ICMP for IPv6.

**Reading a reply line** (e.g. `64 bytes from 10.2.0.x: icmp_seq=3 ttl=127 time=0.8 ms`):

| Part | Meaning |
|---|---|
| `icmp_seq` | Sequence number, matching each reply to its request |
| `ttl` | Time to live: drops by 1 at each router; the packet is discarded at 0 |
| `time` | Round-trip latency in milliseconds |

## Build order and cleanup

1. Create **two VPCs** with unique CIDRs (`10.1.0.0/16`, `10.2.0.0/16`).
2. Create a **peering connection** (VPC 1 = Requester, VPC 2 = Accepter) and **accept** it.
3. Add a **route** to each VPC's route table pointing at the peering connection.
4. Launch an **EC2 instance in each VPC** (edit Network settings to pick the right VPC).
5. Give Instance 1 an **Elastic IP** and allow **SSH (22)** in its security group.
6. Connect with **EC2 Instance Connect** and `ping` Instance 2's private IP.
7. Allow **All ICMP - IPv4** from `10.1.0.0/16` on Instance 2's security group. Ping again and see the replies.

**Cleanup:** terminate both EC2 instances → **release the Elastic IP** (you're charged for public IPv4 addresses) → delete the peering connection → delete both VPCs. If a VPC won't delete, check for leftover **network interfaces** and delete them manually.

---

## Self-quiz

<details><summary>1. What is a VPC peering connection?</summary>

A direct connection between two VPCs that lets their resources communicate using private IP addresses, without going over the public internet.
</details>

<details><summary>2. Why must peered VPCs have unique CIDR blocks?</summary>

If they overlap, route tables can't tell whether traffic should stay in the local VPC or go through the peering connection.
</details>

<details><summary>3. What's the difference between the Requester and the Accepter?</summary>

The Requester sends the peering invitation. The Accepter receives it and must accept it before the connection becomes active.
</details>

<details><summary>4. The peering connection is accepted, but traffic still doesn't flow. What's missing?</summary>

Route table entries. Each VPC needs a route with the other VPC's CIDR as the destination and the peering connection as the target.
</details>

<details><summary>5. A is peered with B, and B is peered with C. Can A reach C?</summary>

No. Peering isn't transitive, so A and C need their own peering connection.
</details>

<details><summary>6. What's the difference between default and dedicated tenancy?</summary>

Default shares hardware with other AWS customers and is cheaper. Dedicated runs on hardware only you use, for compliance or isolation, and costs more.
</details>

<details><summary>7. NAT gateway vs internet gateway?</summary>

An internet gateway allows traffic both ways for public subnets. A NAT gateway lets private subnets reach out to the internet but blocks inbound connections.
</details>

<details><summary>8. What does a VPC endpoint do?</summary>

It connects your VPC privately to AWS services like S3, so traffic stays inside the AWS network instead of the public internet.
</details>

<details><summary>9. Why didn't you need a key pair for these instances?</summary>

EC2 Instance Connect pushes a temporary SSH key for each session, so AWS manages the key for you.
</details>

<details><summary>10. Why couldn't you use EC2 Instance Connect at first, and what fixed it?</summary>

The instance had no public IPv4 address because auto-assign public IP was disabled. Associating an Elastic IP fixed it. Then SSH (port 22) also had to be allowed in the security group.
</details>

<details><summary>11. Why do companies use Elastic IPs?</summary>

The address stays the same across restarts, so DNS records don't need updating. DNS updates can take hours or days to spread, which would mean downtime.
</details>

<details><summary>12. What happens when you release an Elastic IP?</summary>

It goes back to Amazon's pool and may be given to another AWS user later.
</details>

<details><summary>13. You ping Instance 2 and see one line, then nothing. What's likely wrong?</summary>

Instance 2's security group is blocking ICMP traffic. Add an inbound rule for All ICMP - IPv4 from VPC 1's CIDR (`10.1.0.0/16`).
</details>

<details><summary>14. Why is ICMP blocked by default?</summary>

To help prevent attacks that flood a server with ping messages so it can't respond to real users.
</details>

<details><summary>15. In a ping reply, what do TTL and time mean?</summary>

TTL is how many more router hops the packet can make before it's dropped. Time is the round-trip latency in milliseconds.
</details>

<details><summary>16. What should you ping to test the peering connection?</summary>

The **private IPv4 address of Instance 2**, not the VPC ID or the VPC's network address (`10.2.0.0`).
</details>
