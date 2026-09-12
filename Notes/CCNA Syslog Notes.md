# Syslog — CCNA Notes

> **Syslog** is a standard mechanism used by network devices to generate, store, and send messages about system events.

---

## 1. What is Syslog?

Syslog records events that happen on network devices such as:

- Routers
- Switches
- Firewalls
- Servers

Examples of events:

- Interface goes up/down
- Authentication failures
- Configuration changes
- Routing protocol events
- Hardware problems
- Security-related events
- System errors

### Simple idea

```text
Network Device
     │
     │  Syslog messages
     ▼
Syslog Server
     │
     ▼
Store / Analyze / Troubleshoot
```

Think of Syslog as:

> **The network device's event log.**

---

# 2. Syslog Message Format

A Cisco Syslog message can look like:

```text
%LINK-3-UPDOWN: Interface GigabitEthernet0/0, changed state to down
```

Breakdown:

```text
%LINK-3-UPDOWN
 │    │    │
 │    │    └── Mnemonic
 │    └─────── Severity
 └──────────── Facility
```

### Facility

Identifies the subsystem that generated the message.

Example:

```text
LINK
```

means the message is related to the link/interface subsystem.

### Severity

Indicates how serious the event is.

Example:

```text
3
```

means **Error**.

### Mnemonic

A short identifier describing the event.

Example:

```text
UPDOWN
```

---

# 3. Syslog Severity Levels ⭐⭐⭐

There are **8 Syslog severity levels**.

| Level | Name | Description |
|---:|---|---|
| **0** | Emergency | System is unusable |
| **1** | Alert | Immediate action required |
| **2** | Critical | Critical condition |
| **3** | Error | Error condition |
| **4** | Warning | Warning condition |
| **5** | Notification | Normal but significant event |
| **6** | Informational | Informational message |
| **7** | Debugging | Debugging messages |

### Memorize

```text
0 Emergency
1 Alert
2 Critical
3 Error
4 Warning
5 Notification
6 Informational
7 Debugging
```

### Important rule

> **Lower number = higher severity**

```text
MOST SEVERE
     │
     ▼
0  Emergency
1  Alert
2  Critical
3  Error
4  Warning
5  Notification
6  Informational
7  Debugging
     ▲
     │
LEAST SEVERE
```

---

# 4. Severity Filtering ⭐⭐⭐

When you configure a Syslog severity level, the device sends that level **and all more severe levels**.

For example:

```text
logging trap 4
```

means:

```text
0 Emergency
1 Alert
2 Critical
3 Error
4 Warning
```

It does **NOT** mean only level 4.

### Examples

```text
logging trap 0
```

Sends:

```text
0
```

---

```text
logging trap 3
```

Sends:

```text
0
1
2
3
```

---

```text
logging trap 5
```

Sends:

```text
0
1
2
3
4
5
```

---

# 5. Syslog Transport Protocol ⭐⭐⭐

Traditional Syslog commonly uses:

```text
UDP 514
```

### Remember

```text
Syslog → UDP → Port 514
```

Because UDP is connectionless, traditional Syslog does not provide TCP-style delivery guarantees.

---

# 6. Where Cisco IOS Can Send/Store Logs

Cisco IOS can send logging information to several destinations.

```text
                    Cisco Device
                         │
             ┌───────────┼───────────┐
             │           │           │
             ▼           ▼           ▼
          Console      Buffer      Syslog
                                  Server
             │           │           │
             ▼           ▼           ▼
          Display       RAM       Remote
```

Common destinations:

1. Console
2. Terminal/VTY
3. Logging buffer
4. Remote Syslog server

---

# 7. Console Logging

Console logging displays Syslog messages on the console.

```text
Router(config)# logging console
```

You can specify a severity:

```text
Router(config)# logging console warnings
```

`warnings` = severity **4**

Therefore:

```text
0–4
```

will be displayed.

---

# 8. Buffered Logging

Cisco can store logs in a buffer in RAM.

```text
Router(config)# logging buffered
```

You can specify the buffer size:

```text
Router(config)# logging buffered 16384
```

View the logs:

```text
Router# show logging
```

### Important

> **Buffered logs are stored in RAM.**

Therefore, they are normally lost after a device reboot.

---

# 9. Terminal Logging

If connected through SSH or Telnet, use:

```text
Router# terminal monitor
```

This allows Syslog messages to appear in your current terminal session.

Disable it with:

```text
Router# terminal no monitor
```

### Remember

```text
terminal monitor
```

= Show Syslog messages in the current VTY session.

---

# 10. `logging synchronous`

This command helps prevent Syslog messages from disrupting your CLI input.

Configure it under the line configuration:

```text
Router(config)# line console 0
Router(config-line)# logging synchronous
```

For VTY lines:

```text
Router(config)# line vty 0 4
Router(config-line)# logging synchronous
```

### Don't confuse these commands

```text
terminal monitor
```

→ Display Syslog messages in your terminal.

```text
logging synchronous
```

→ Keep the CLI display organized when Syslog messages appear.

---

# 11. Remote Syslog Server ⭐⭐⭐

A network device can send logs to a remote Syslog server.

Example:

```text
Syslog Server = 192.168.1.100
```

Configuration:

```text
Router(config)# logging 192.168.1.100
```

Some IOS versions/contexts also use:

```text
Router(config)# logging host 192.168.1.100
```

The idea is:

```text
Router
   │
   │ UDP 514
   ▼
192.168.1.100
Syslog Server
```

---

# 12. Configure Remote Syslog Severity

Example:

```text
Router(config)# logging trap 4
```

This sends severity:

```text
0 Emergency
1 Alert
2 Critical
3 Error
4 Warning
```

to the remote Syslog server.

---

# 13. Complete Example Configuration

Suppose:

```text
Syslog Server = 10.10.10.100
```

Configure:

```text
Router(config)# logging 10.10.10.100
Router(config)# logging trap 5
Router(config)# logging buffered 16384
Router(config)# service timestamps log datetime
```

If connected through SSH:

```text
Router# terminal monitor
```

Check the configuration/logs:

```text
Router# show logging
```

---

# 14. Syslog Timestamps ⭐⭐⭐

Timestamps help determine exactly when an event happened.

Enable them with:

```text
Router(config)# service timestamps log datetime
```

A log message can then include a timestamp such as:

```text
Sep 12 10:30:15: %LINK-3-UPDOWN:
Interface GigabitEthernet0/0, changed state to down
```

### Why timestamps matter

They allow administrators to build an accurate timeline:

```text
10:15 → Interface went down
10:16 → Routing neighbor went down
10:20 → Interface came back up
```

---

# 15. Syslog and NTP

Syslog timestamps are much more useful when the device has an accurate clock.

NTP synchronizes the device clock.

```text
NTP
 │
 ▼
Accurate device time
 │
 ▼
Syslog timestamps
 │
 ▼
Accurate event timeline
```

Remember:

> **NTP = synchronize time**

> **Syslog = record events**

---

# 16. `show logging` ⭐⭐⭐

Use:

```text
Router# show logging
```

to inspect:

- Logging status
- Console logging
- Monitor logging
- Buffer logging
- Remote Syslog servers
- Severity levels
- Logged messages

Example:

```text
Router# show logging
```

This is one of the most important Syslog troubleshooting commands for CCNA.

---

# 17. Example: Interface Goes Down

Suppose an interface goes down:

```text
Router(config)# interface g0/0
Router(config-if)# shutdown
```

Cisco may generate a message similar to:

```text
%LINK-5-CHANGED: Interface GigabitEthernet0/0,
changed state to administratively down
```

Break it down:

```text
LINK
 │
 └── Facility

5
 │
 └── Severity = Notification

CHANGED
 │
 └── Mnemonic
```

Because severity is **5**, this message is sent to a remote Syslog server only if the configured trap level includes level 5.

For example:

```text
logging trap 4
```

❌ Level 5 is NOT included.

But:

```text
logging trap 5
```

✅ Level 5 IS included.

---

# 18. Syslog vs SNMP

These are easy to confuse.

### Syslog

Primarily used for:

> **Event/log messages**

Examples:

```text
Interface went down
Configuration changed
Authentication failed
Routing event occurred
```

### SNMP

Primarily used for:

> **Network monitoring and management**

Examples:

```text
CPU utilization
Memory utilization
Interface statistics
Device status
```

Simple mental model:

```text
Syslog → "What happened?"
SNMP   → "How is the device doing?"
```

---

# 19. Syslog vs NTP

| Technology | Purpose |
|---|---|
| **Syslog** | Records system events |
| **NTP** | Synchronizes time |

They work well together:

```text
NTP
 ↓
Correct time
 ↓
Syslog
 ↓
Accurate timestamps
```

---

# 20. Important Cisco Commands

| Command | Purpose |
|---|---|
| `show logging` | View logging information |
| `logging <IP>` | Configure remote Syslog server |
| `logging host <IP>` | Configure remote Syslog server |
| `logging trap <level>` | Set remote Syslog severity |
| `logging buffered` | Store logs in RAM |
| `logging console` | Enable console logging |
| `terminal monitor` | Display logs in VTY session |
| `terminal no monitor` | Disable terminal monitoring |
| `logging synchronous` | Prevent logs from disrupting CLI |
| `service timestamps log datetime` | Add timestamps |

---

# 21. CCNA Must-Know ⭐⭐⭐⭐⭐

If you're short on study time, memorize these:

### Syslog basics

```text
Syslog = event logging
```

### Transport

```text
Syslog = UDP 514
```

### Severity

```text
0 Emergency
1 Alert
2 Critical
3 Error
4 Warning
5 Notification
6 Informational
7 Debugging
```

### Severity rule

```text
Lower number = more severe
```

### Filtering

```text
logging trap 4
```

means:

```text
0–4
```

NOT only level 4.

### View logs

```text
show logging
```

### Remote server

```text
logging 192.168.1.100
```

### VTY session

```text
terminal monitor
```

### RAM buffer

```text
logging buffered
```

### Timestamps

```text
service timestamps log datetime
```

---

# 22. CCNA Exam Questions

## Question 1

A router is configured with:

```text
R1(config)# logging trap 4
```

Which severity levels are sent to the remote Syslog server?

- A. Only 4
- B. 4–7
- C. 0–4
- D. Only 0

### Answer

**C. 0–4**

Because lower numbers represent more severe events.

---

## Question 2

Which protocol and port are commonly associated with traditional Syslog?

- A. TCP 514
- B. UDP 514
- C. TCP 161
- D. UDP 162

### Answer

**B. UDP 514**

---

## Question 3

An administrator is connected to a router using SSH and wants to see Syslog messages in the terminal.

Which command should be used?

```text
R1# terminal monitor
```

### Answer

**`terminal monitor`**

---

## Question 4

Which command displays logging information on a Cisco router?

```text
R1# show logging
```

### Answer

**`show logging`**

---

## Question 5

Which severity is the most serious?

- A. 7
- B. 5
- C. 3
- D. 0

### Answer

**D. 0 — Emergency**

---

# 23. Quick Memory Map

```text
                         SYSLOG
                            │
             ┌──────────────┼──────────────┐
             │              │              │
           WHAT?          HOW?           WHERE?
             │              │              │
          Events       Severity 0–7    Console
                            │          Buffer
                            │          Terminal
                            │          Server
                            │
                         UDP 514
```

### Severity

```text
0 → Emergency       ← Most severe
1 → Alert
2 → Critical
3 → Error
4 → Warning
5 → Notification
6 → Informational
7 → Debugging       ← Least severe
```

### Core commands

```text
logging <IP>
logging trap <level>
logging buffered
terminal monitor
show logging
service timestamps log datetime
```

---

# 24. Final CCNA Syslog Cheat Sheet

```text
┌─────────────────────────────────────────────┐
│                 SYSLOG                      │
├─────────────────────────────────────────────┤
│ Purpose:        Record system events        │
│ Transport:      UDP                         │
│ Port:           514                         │
├─────────────────────────────────────────────┤
│ 0 Emergency                                 │
│ 1 Alert                                     │
│ 2 Critical                                  │
│ 3 Error                                     │
│ 4 Warning                                   │
│ 5 Notification                              │
│ 6 Informational                             │
│ 7 Debugging                                 │
├─────────────────────────────────────────────┤
│ Lower number = More severe                  │
│ logging trap 4 = Levels 0–4                 │
├─────────────────────────────────────────────┤
│ logging <IP>            → Syslog server    │
│ logging trap <level>    → Severity filter  │
│ logging buffered        → RAM buffer       │
│ terminal monitor        → VTY log display  │
│ show logging            → View logs        │
│ logging synchronous     → Clean CLI output │
│ service timestamps...   → Log timestamps   │
└─────────────────────────────────────────────┘
```

## 🎯 One thing to remember

> **Syslog tells you WHAT happened, severity tells you HOW serious it is, and the Syslog server gives you a central place to store the events.**