# Linux Packet Filter using Netfilter

A simple Linux kernel firewall project built with the **Netfilter framework**.

The project filters incoming IPv4 packets in kernel space and allows firewall rules to be managed dynamically from user space through a command-line interface. Communication between the CLI and the kernel module is implemented using **Netlink sockets**.

## Features

- Filter IPv4 packets using Linux Netfilter
- Block packets from multiple source IP addresses
- Remove blocked IP addresses
- Apply per-IP traffic rate limiting
- Update rate-limit rules dynamically
- Remove rate-limit rules
- Display current firewall rules
- Communicate between user space and kernel space using Netlink
- Log dropped packets in the Linux kernel log

## Architecture

```text
User Space
+-----------------------+
|        fw_cli         |
|                       |
| block add / del       |
| ratelimit add / del   |
| show                  |
+----------+------------+
           |
           | Netlink Socket
           |
+----------v------------+
|   Linux Kernel Module |
|                       |
|   Block IP List       |
|   Rate-limit IP List  |
+----------+------------+
           |
           | Netfilter Hook
           | NF_INET_PRE_ROUTING
           |
+----------v------------+
|    IPv4 Packet        |
|                       |
|    ACCEPT / DROP      |
+-----------------------+
```

The kernel module registers a Netfilter hook at the `NF_INET_PRE_ROUTING` stage.

For each IPv4 packet:

1. The source IP is checked against the block list.
2. If the source IP is blocked, the packet is dropped.
3. Otherwise, the source IP is checked against the rate-limit list.
4. Traffic from that IP is counted in a fixed time window of approximately one second.
5. If the configured byte limit is exceeded, the packet is dropped.
6. All other packets are accepted.

## Project Structure

```text
linux-packet-filter-netfilter/
├── fw_netlink_kmod.c   # Kernel module and Netfilter logic
├── fw_cli.c            # User-space CLI for rule management
├── Makefile            # Kernel module build configuration
└── README.md
```

## Requirements

- Linux operating system
- GCC
- Make
- Linux kernel headers

On Ubuntu/Debian:

```bash
sudo apt update
sudo apt install build-essential linux-headers-$(uname -r)
```

## Build

Before building, make sure the first line of the `Makefile` is:

```makefile
obj-m += fw_netlink_kmod.o
```

Compile the kernel module:

```bash
make
```

Compile the CLI:

```bash
gcc fw_cli.c -o fw_cli
```

## Load the Kernel Module

```bash
sudo insmod fw_netlink_kmod.ko
```

Check whether the module is loaded:

```bash
lsmod | grep fw_netlink_kmod
```

View kernel logs:

```bash
sudo dmesg -w
```

## CLI Usage

### Show current rules

```bash
sudo ./fw_cli show
```

### Block an IP address

```bash
sudo ./fw_cli block add 192.168.1.100
```

### Remove an IP from the block list

```bash
sudo ./fw_cli block del 192.168.1.100
```

### Add or update a rate-limit rule

```bash
sudo ./fw_cli ratelimit add 192.168.1.100 10000
```

The final argument is the allowed traffic amount in **bytes per second**.

Example:

```text
10000 B/s
```

### Remove a rate-limit rule

```bash
sudo ./fw_cli ratelimit del 192.168.1.100
```

## Packet Processing Flow

```text
IPv4 Packet
     |
     v
Check Block List
     |
 +---+---+
 |       |
Yes      No
 |       |
DROP     v
      Check Rate Limit
           |
       +---+---+
       |       |
    Exceed   Allow
       |       |
      DROP   ACCEPT
```

## Technologies

- C
- Linux Kernel Module
- Netfilter
- Netlink Socket
- Linux Linked List
- Spinlock
- IPv4 Networking

## Learning Objectives

This project was developed to gain practical experience with:

- Linux kernel programming
- Packet processing in the Linux networking stack
- Netfilter hooks
- Kernel-space and user-space communication
- Netlink sockets
- Dynamic firewall rule management
- Basic packet filtering
- Traffic rate limiting

## Current Limitations

- IPv4 only
- Rules are based on source IP addresses
- No TCP/UDP port filtering
- No protocol-specific filtering
- Rate limiting uses a simple fixed-window mechanism
- Rules are not persistent after the kernel module is unloaded

## Future Improvements

- Add TCP/UDP port filtering
- Add protocol-based rules
- Add IPv6 support
- Add persistent rule storage
- Improve the rate-limiting algorithm
- Add packet and rule statistics
- Improve CLI usability

## Author

**MINHDANGab**
