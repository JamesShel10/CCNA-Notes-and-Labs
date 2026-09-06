# ACLs - Access Control Lists

## 1. What is an ACL?

An **Access Control List (ACL)** is a set of rules used on network devices to control traffic.

ACLs can be used to:

* Permit traffic
* Deny traffic
* Control access to network resources
* Filter packets
* Improve network security

---

## 2. Types of ACLs

There are two main types of IPv4 ACLs:

### Standard ACL

Standard ACLs filter traffic based only on the **source IPv4 address**.

```text
Source IP → Permit/Deny
```

Standard ACLs are numbered:

```text
1-99
1300-1999
```

Example:

```cisco
access-list 10 permit 192.168.1.0 0.0.0.255
```

---

### Extended ACL

Extended ACLs provide more detailed control.

They can filter based on:

* Source IP address
* Destination IP address
* Protocol
* TCP/UDP
* Source port
* Destination port

Extended ACLs are numbered:

```text
100-199
2000-2699
```

Example:

```cisco
access-list 100 permit tcp 192.168.1.0 0.0.0.255 any eq 80
```

This permits HTTP traffic from the `192.168.1.0/24` network.

---

# 3. Wildcard Masks

ACLs commonly use **wildcard masks**.

A wildcard mask is the inverse of a subnet mask.

Example:

```text
Subnet mask:   255.255.255.0
Wildcard mask: 0.0.0.255
```

Another example:

```text
Subnet mask:   255.255.255.252
Wildcard mask: 0.0.0.3
```

### Common Wildcard Masks

| Network | Subnet Mask     | Wildcard Mask |
| ------- | --------------- | ------------- |
| /24     | 255.255.255.0   | 0.0.0.255     |
| /25     | 255.255.255.128 | 0.0.0.127     |
| /26     | 255.255.255.192 | 0.0.0.63      |
| /27     | 255.255.255.224 | 0.0.0.31      |
| /28     | 255.255.255.240 | 0.0.0.15      |
| /30     | 255.255.255.252 | 0.0.0.3       |

---

# 4. Standard ACL Configuration

Example requirement:

> Allow network `192.168.1.0/24`.

```cisco
Router(config)# access-list 10 permit 192.168.1.0 0.0.0.255
```

Apply the ACL to an interface:

```cisco
Router(config)# interface gigabitEthernet 0/0
Router(config-if)# ip access-group 10 in
```

---

# 5. Extended ACL Configuration

Example requirement:

> Allow HTTP traffic from `192.168.1.0/24` to any destination.

```cisco
Router(config)# access-list 100 permit tcp 192.168.1.0 0.0.0.255 any eq 80
```

Apply it:

```cisco
Router(config)# interface gigabitEthernet 0/0
Router(config-if)# ip access-group 100 in
```

---

# 6. Named ACLs

Named ACLs are easier to understand and modify.

Example:

```cisco
Router(config)# ip access-list extended WEB-ACCESS
Router(config-ext-nacl)# permit tcp 192.168.1.0 0.0.0.255 any eq 80
Router(config-ext-nacl)# deny ip any any
Router(config-ext-nacl)# exit
```

Apply it:

```cisco
Router(config)# interface gigabitEthernet 0/0
Router(config-if)# ip access-group WEB-ACCESS in
```

---

# 7. Implicit Deny

Every ACL has an **implicit deny** at the end.

For example:

```cisco
access-list 10 permit 192.168.1.0 0.0.0.255
```

Conceptually behaves like:

```text
permit 192.168.1.0/24
deny everything else
```

Therefore, if traffic does not match a permit statement, it is denied.

---

# 8. ACL Processing Rules

ACLs are processed:

1. From **top to bottom**
2. One statement at a time
3. The first matching statement is used
4. Processing stops after a match

Example:

```cisco
access-list 100 deny ip host 192.168.1.10 any
access-list 100 permit ip 192.168.1.0 0.0.0.255 any
```

The host `192.168.1.10` is denied because the first rule matches.

---

# 9. Inbound vs Outbound ACLs

### Inbound

Traffic is checked when it **enters** the interface.

```cisco
ip access-group 100 in
```

### Outbound

Traffic is checked when it **leaves** the interface.

```cisco
ip access-group 100 out
```

---

# 10. Useful ACL Commands

Show ACLs:

```cisco
show access-lists
```

Show IPv4 ACLs:

```cisco
show ip access-lists
```

Show interfaces and ACLs:

```cisco
show ip interface
```

Show running configuration:

```cisco
show running-config
```

---

# 11. Removing an ACL

Remove a numbered ACL:

```cisco
no access-list 10
```

Remove an ACL from an interface:

```cisco
interface gigabitEthernet 0/0
no ip access-group 10 in
```

---

# 12. Important CCNA Points

Remember:

* **Standard ACL → source IPv4 address**
* **Extended ACL → source, destination, protocol and ports**
* ACLs are processed **top to bottom**
* **First match wins**
* There is an **implicit deny** at the end
* Standard ACLs should generally be placed closer to the **destination**
* Extended ACLs should generally be placed closer to the **source**
* Wildcard masks are used to identify addresses/ranges
* ACLs can be applied **inbound** or **outbound**

---

# Quick Revision

| Concept          | Remember                                |
| ---------------- | --------------------------------------- |
| Standard ACL     | Source IP                               |
| Extended ACL     | Source + Destination + Protocol + Ports |
| Standard numbers | 1-99, 1300-1999                         |
| Extended numbers | 100-199, 2000-2699                      |
| Processing       | Top → Bottom                            |
| Matching         | First match                             |
| End of ACL       | Implicit deny                           |
| Inbound          | Entering interface                      |
| Outbound         | Leaving interface                       |
| Wildcard /24     | 0.0.0.255                               |
