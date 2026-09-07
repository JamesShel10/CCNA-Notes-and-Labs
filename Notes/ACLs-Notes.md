# Access Control Lists (ACLs)

## Table of Contents
1. [What are ACLs?](#1-what-are-acls)
2. [How ACLs Work (ACL Logic)](#2-how-acls-work-acl-logic)
3. [Implicit Deny](#3-implicit-deny)
4. [ACL Types](#4-acl-types)
5. [Standard ACLs](#5-standard-acls)
   - [Standard Numbered ACLs](#51-standard-numbered-acls)
   - [Standard Named ACLs](#52-standard-named-acls)
6. [Extended ACLs](#6-extended-acls)
   - [Extended Numbered ACLs](#61-extended-numbered-acls)
   - [Extended Named ACLs](#62-extended-named-acls)
7. [Verification Commands](#7-verification-commands)
8. [Best Practices](#8-best-practices)

---

## 1) What are ACLs?

- **ACL** stands for **Access Control List**.
- An ACL acts as a **packet filter**, instructing the router to **permit** or **deny** specific traffic.
- ACLs can filter traffic based on:
  - Source / destination **IP address**
  - Source / destination **Layer 4 port** (TCP/UDP)

---

## 2) How ACLs Work (ACL Logic)

- ACLs are defined in **global configuration** but have **no effect** until applied to an interface.
- An ACL is applied to an interface in one of two directions:
  - **Inbound** — filters traffic entering the interface
  - **Outbound** — filters traffic leaving the interface
- An ACL is an **ordered sequence of ACEs** (Access Control Entries).
- **Order matters** — the router processes entries **top to bottom** and stops at the first match.

<img width="389" height="125" alt="ACL processing diagram" src="https://github.com/user-attachments/assets/02938648-d108-48d4-9684-7766a95f9b3c" />

### Example Scenario

**Requirement:**
- `192.168.1.0/24` **can** access `10.0.1.0/24` (SRV1)
- `192.168.2.0/24` **cannot** access `10.0.1.0/24` (SRV1)

**ACL1:**
```
A: permit 192.168.1.0/24
B: deny   192.168.2.0/24
C: permit any
```

**Topology logic:**
PC3 (`192.168.2.1`) sends a packet to its gateway `R1 (G0/2)`. The packet enters R1 through `G0/2` (**inbound**), is routed, and leaves through `G0/0` toward `R2`, which forwards it out `G0/1` to `SRV1`.

> ⚠️ Applying **ACL1 outbound** on `R1 G0/2` will **not** work — outbound ACLs filter traffic *leaving* an interface, but PC3's traffic is *entering* R1 on that interface, not leaving it. This configuration is effectively meaningless for this requirement.

To achieve the requirement, ACL1 must be applied **inbound** on `R1 G0/2`. R1 will then evaluate the ACL top to bottom against traffic entering that interface.

> ⚠️ **Trade-off:** Applying an inbound ACL on `G0/2` blocks matching traffic to **all** destinations reachable through R1, not just SRV1. This may over-restrict legitimate traffic — so interface/direction placement must be chosen carefully based on the actual requirement.

**Key design question:** *Where is the best place to apply this ACL — and in which direction — so it blocks only the intended traffic?* (See [Best Practices](#8-best-practices) below.)

---

## 3) Implicit Deny

- Every ACL ends with an **implicit deny all**, even if not explicitly configured.
- If a packet doesn't match **any** entry in the ACL, it is dropped.

**Example — ACL2:**
```
1) permit 192.168.1.0/24
2) deny   192.168.0.0/16
3) deny   any            (implicit)
```

If a packet has source `10.0.0.1` and destination `1.1.1.1`, it doesn't match entries 1 or 2, so it falls through to the implicit deny and is **dropped**.

---

## 4) ACL Types

| Type | Matches Traffic Based On |
|------|---------------------------|
| **Standard ACL** | Source IP address only |
| **Extended ACL** | Source/destination IP address **and** source/destination Layer 4 port |

Each type comes in two configuration styles: **Numbered** and **Named**.

---

## 5) Standard ACLs

- Match traffic based on **source IP address only**.
- Numbered range: **1–99** and **1300–1999**.

### 5.1) Standard Numbered ACLs

**Syntax:**
```
R1(config)# access-list <number> {deny | permit} <source-ip> [wildcard-mask]
```

**Example:**
```
R1(config)# access-list 1 deny 1.1.1.1 0.0.0.0
R1(config)# access-list 1 deny 1.1.1.1
R1(config)# access-list 1 permit any
```

**Adding a remark (documentation only, no filtering effect):**
```
R1(config)# access-list 1 remark ## BLOCK BOB FROM ACCOUNTING DEPT ##
```

**Applying the ACL to an interface:**
```
R1(config)# interface g0/0
R1(config-if)# ip access-group 1 {in | out}
```

> 📌 **Note:** Standard ACLs should be applied **as close to the destination as possible**, since they can only filter by source IP and can't distinguish traffic beyond that.

**Verification:**
```
R1# show access-lists
R1# show running-config | include access-list
```

### 5.2) Standard Named ACLs

**Syntax:**
```
R1(config)# ip access-list standard <acl-name>
R1(config-std-nacl)# [entry-number] {deny | permit} <ip> [wildcard-mask]
```

**Example:**
```
R1(config)# ip access-list standard BLOCK_BOB
R1(config-std-nacl)# 5 deny 1.1.1.1
R1(config-std-nacl)# 10 permit any
R1(config-std-nacl)# remark ## CONFIGURED NOV 21 2020 ##
R1(config-std-nacl)# exit
R1(config)# interface g0/0
R1(config-if)# ip access-group BLOCK_BOB in
```

**Verification:**
```
R1# show access-lists
R1# show running-config | section access-list
```

---

## 6) Extended ACLs

- Match traffic based on **source/destination IP address** and **source/destination port** (protocol-aware: TCP, UDP, ICMP, etc.).
- Numbered range: **100–199** and **2000–2699**.
- Because they can match very specifically, extended ACLs should be applied **as close to the source as possible**, to avoid wasting bandwidth carrying traffic through the network only to drop it later.

### 6.1) Extended Numbered ACLs

**Syntax:**
```
R1(config)# access-list <number> {deny | permit} <protocol> <source> [source-wildcard] [operator port] <destination> [destination-wildcard] [operator port]
```

**Example — block PC (192.168.1.10) from reaching a web server (10.0.1.5) on HTTP, permit everything else:**
```
R1(config)# access-list 100 deny tcp host 192.168.1.10 host 10.0.1.5 eq 80
R1(config)# access-list 100 permit ip any any
```

**Applying to an interface:**
```
R1(config)# interface g0/1
R1(config-if)# ip access-group 100 in
```

**Verification:**
```
R1# show access-lists
R1# show running-config | include access-list
```

### 6.2) Extended Named ACLs

**Syntax:**
```
R1(config)# ip access-list extended <acl-name>
R1(config-ext-nacl)# [entry-number] {deny | permit} <protocol> <source> [wildcard] [operator port] <destination> [wildcard] [operator port]
```

**Example:**
```
R1(config)# ip access-list extended BLOCK_HTTP_TO_SRV1
R1(config-ext-nacl)# 10 deny tcp host 192.168.1.10 host 10.0.1.5 eq 80
R1(config-ext-nacl)# 20 permit ip any any
R1(config-ext-nacl)# remark ## CONFIGURED SEP 07 2026 ##
R1(config-ext-nacl)# exit
R1(config)# interface g0/1
R1(config-if)# ip access-group BLOCK_HTTP_TO_SRV1 in
```

**Verification:**
```
R1# show access-lists
R1# show running-config | section access-list
```

---

## 7) Verification Commands

| Command | Purpose |
|---------|---------|
| `show access-lists` | Displays all configured ACLs and match/hit counters |
| `show ip interface <interface>` | Shows which ACLs are applied to an interface and in which direction |
| `show running-config \| include access-list` | Filters running-config for ACL-related lines |
| `show running-config \| section access-list` | Shows full ACL configuration blocks |

---

## 8) Best Practices

- **Standard ACLs** → apply close to the **destination** (they can only match source IP, so applying them near the source risks blocking traffic that should reach other legitimate destinations).
- **Extended ACLs** → apply close to the **source** (since they can match specifically enough to avoid unnecessary blocking of unrelated traffic).
- Always remember the **implicit deny** at the end of every ACL — add `permit any` (or `permit ip any any` for extended) explicitly if you don't want to unintentionally block all remaining traffic.
- **Order entries carefully** — more specific rules should generally come before more general ones.
- Use **remarks** to document the intent of ACL entries for future troubleshooting.
- Only **one ACL per protocol, per direction, per interface** can be applied at a time.
