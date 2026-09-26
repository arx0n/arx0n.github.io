# CIDrift

CIDrift is a small Bash tool designed to calculate IPv4 network information from an IP address using CIDR notation.

The idea came up to make auditing tasks easier. For example, if I'm told that I have to audit this network → `10.10.5.1/13`, I can quickly determine which IP range I should be working with and get an idea of how many hosts there could be.

## What does it calculate?

CIDrift can calculate:

- **Network Mask**
- **Network ID**
- **Broadcast Address**
- **Usable Hosts**

The tool also performs basic validation of the IPv4 address and the specified CIDR prefix.

## Usage

You can use CIDrift directly from the terminal:

```bash
cidrift 192.168.1.10/24
```

You can also run it without arguments and provide the IP address when prompted:

```bash
cidrift
```

Example:

```text
[+] IP Address -> 192.168.1.10/24
[+] Total hosts -> 254
[+] Network Mask -> 255.255.255.0
[+] Network ID -> 192.168.1.0
[+] Broadcast Address -> 192.168.1.255
```

## Installation

You can find the source code and installer on GitHub:

[**Cidrift - GitHub**](https://github.com/arx0n/cidrift)

To install it:

```bash
git clone https://github.com/arx0n/cidrift.git
cd cidrift
sudo ./install.sh
```

Once installed, you can use CIDrift from anywhere:

```bash
cidrift 192.168.1.10/24
```

## Features

- IPv4
- CIDR notation
- Input validation
- Network mask calculation
- Network ID calculation
- Broadcast address calculation
- Usable host calculation
- Terminal interface
- No external dependencies

## Source Code

CIDrift is an open-source project, and its source code is available on GitHub.

[**GitHub → arx0n/cidrift**](https://github.com/arx0n/cidrift)

---

**CIDrift · Made by arx0n**
