# Extended ACLs

## Topics Covered
1. Another way to configure numbered ACLs
2. Editing ACLs
3. Extended numbered and named ACLs

---

## 1. Named vs. Numbered ACL Configuration

### Advantages of Named ACL Config Mode
- When configuring/editing **numbered ACLs** from global config mode, you **can't delete individual entries** — you can only delete the entire ACL.
- With **named ACLs** (or numbered ACLs edited in ACL config submode), you can easily delete **individual entries** using their sequence number.
- You can **insert new entries between existing ones** by specifying a sequence number.

### Resequencing ACLs
Resequencing helps you edit ACLs — for example, when you want to insert a new entry between existing entries but there isn't enough sequence-number "room" to do so.

```
R1(config)# ip access-list resequence <acl-id> <starting-seq-num> <increment>
```

**Example:**
```
R1(config)# ip access-list resequence 1 10 10
```
- First `10` → sets the sequence number of the **first entry** to 10
- Second `10` → the **increment** added to each subsequent entry (20, 30, 40, ...)

---

## 2. Extended ACLs

- Numbered extended ACLs use the ranges: **100–199** and **2000–2699**
- Processed **top to bottom** (first match wins)
- Can match traffic on many more parameters than standard ACLs, so they are more precise (and more complex)

### Basic Syntax

```
R1(config)# access-list <number> [permit | deny] <protocol> <src-ip> <dest-ip>
```

Named/sequenced form (entered from ACL config submode):
```
R1(config-ext-nacl)# [seq-num] [permit | deny] <protocol> <src-ip> <dest-ip>
```

### Matching the Protocol
Can be specified by number or name:

| Number | Protocol |
|--------|----------|
| 1      | ICMP     |
| 6      | TCP      |
| 17     | UDP      |
| 88     | EIGRP    |
| 89     | OSPF     |

### Matching Source/Destination IP Address
- `any` — matches all addresses
- `host <ip>` — matches a single address (equivalent to a /32 wildcard mask of `0.0.0.0`)
- `<network> <wildcard-mask>` — matches a range/subnet (e.g., `10.0.0.0 0.0.255.255` for 10.0.0.0/16)

---

## Extended ACL Entry Practice (1)

**1) Allow all traffic**
```
R1(config-ext-nacl)# permit ip any any
```

**2) Prevent 10.0.0.0/16 from sending UDP traffic to 192.168.1.1/32**
```
R1(config-ext-nacl)# deny udp 10.0.0.0 0.0.255.255 host 192.168.1.1
```

**3) Prevent 172.168.1.1/32 from pinging hosts in 192.168.0.0/24**
```
R1(config-ext-nacl)# deny icmp host 172.168.1.1 192.168.0.0 0.0.0.255
```

---

## Matching TCP/UDP Port Numbers

When matching TCP or UDP, you can optionally specify source and/or destination port numbers.

```
R1(config-ext-nacl)# deny tcp <src-ip> <dest-ip> eq <port-num>
```

| Operator      | Meaning                          |
|---------------|-----------------------------------|
| `eq 80`       | equal to port 80                 |
| `gt 80`       | greater than 80 (81+)             |
| `lt 80`       | less than 80 (79 and below)       |
| `neq 80`      | not equal to 80                   |
| `range 80 100`| from port 80 to port 100 inclusive|

### Additional Match Options (beyond CCNA scope, but good to know)
After matching source/destination IP and port, there are additional options available:
- `ack` — match the TCP ACK flag
- `fin` — match the TCP FIN flag
- `syn` — match the TCP SYN flag
- `ttl` — match packets with a specific TTL value
- `dscp` — match packets with a specific DSCP value

> **Important:** If you specify protocol, source IP, source port, destination IP, destination port, etc., a packet must match **all** of those values to match that ACL entry. Even if it matches all but one parameter, the packet does **not** match that entry — it continues down the ACL to the next entry.

---

## Extended ACL Entry Practice (2)

**1) Allow all traffic from 10.0.0.0/16 to access the server at 2.2.2.2/32 using HTTPS**
```
R1(config-ext-nacl)# permit tcp 10.0.0.0 0.0.255.255 host 2.2.2.2 eq 443
```

**2) Prevent all hosts using source UDP port numbers 20000–30000 from accessing the server at 3.3.3.3/32**
```
R1(config-ext-nacl)# deny udp any range 20000 30000 host 3.3.3.3
```

**3) Allow hosts in 172.168.1.0/24 using a TCP source port greater than 9999 to access all TCP ports on server 4.4.4.4/32, except port 23**
```
R1(config-ext-nacl)# permit tcp 172.168.1.0 0.0.0.255 gt 9999 host 4.4.4.4 neq 23
```

---

## Quick Reference Summary

| Concept | Command |
|---|---|
| Resequence ACL | `ip access-list resequence <acl-id> <start> <increment>` |
| Extended numbered ACL range | 100–199, 2000–2699 |
| Permit all IP traffic | `permit ip any any` |
| Match a single host | `host <ip>` |
| Match a subnet | `<network> <wildcard-mask>` |
| Match a port | `eq / gt / lt / neq / range <port>` |
