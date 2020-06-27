+++
title = "Scapy p.10"
description = "Emulating nmap Functions"
#date = 2013-10-29
date = 2019-05-11
author = "Mat"
weight = 91

aliases = ["/scapy-p-10-emulating-nmap-functions/"]
[taxonomies]
tags = ["scapy", "python"]
+++

We've seen a lot of cool applications for scapy in your network tools, but a good inspiration for new tools is to look at existing tools to figure out how they do their job. We will be emulating some nmap & Angry IP Scanner type features and creating the following tools:

  * [TCP Port Range Scanner](#port-scanner)
  * [ICMP Ping Sweep](#ping-sweep)
  * [Combining the Two](#sweep-and-scan)

<!-- more -->

#### TCP Port Range Scanner {#port-scanner}

This is a fairly basic tool to test whether a host has specific TCP ports open and listening. We start out by defining our host and ports to scan and then move on to the fun stuff. Using a random TCP source port to help obfuscate the attack (although most firewalls are smarter than this nowadays), we send a TCP SYN packet to each destination TCP port specified. If we get no response or a TCP RST in return, we know that the host is filtering or not listening on that port. If we get an ICMP unreachable or error response, we also know the host is not willing to take requests on that port. But, if we get an expected TCP SYN/ACK response, we will send a RST so the host doesn't keep listening for our ACK since we already know the host is listening on that port. Here's the code:

```python
#! /usr/bin/env python3

import random
from scapy.all import ICMP, IP, sr1, TCP

# Define end host and TCP port range
host = "192.168.40.1"
port_range = [22, 23, 80, 443, 3389]

# Send SYN with random Src Port for each Dst port
for dst_port in port_range:
    src_port = random.randint(1025,65534)
    resp = sr1(
        IP(dst=host)/TCP(sport=src_port,dport=dst_port,flags="S"),timeout=1,
        verbose=0,
    )

    if resp is None:
        print(f"{host}:{dst_port} is filtered (silently dropped).")

    elif(resp.haslayer(TCP)):
        if(resp.getlayer(TCP).flags == 0x12):
            # Send a gratuitous RST to close the connection
            send_rst = sr(
                IP(dst=host)/TCP(sport=src_port,dport=dst_port,flags='R'),
                timeout=1,
                verbose=0,
            )
            print(f"{host}:{dst_port} is open.")

        elif (resp.getlayer(TCP).flags == 0x14):
            print(f"{host}:{dst_port} is closed.")

    elif(resp.haslayer(ICMP)):
        if(
            int(resp.getlayer(ICMP).type) == 3 and
            int(resp.getlayer(ICMP).code) in [1,2,3,9,10,13]
        ):
            print(f"{host}:{dst_port} is filtered (silently dropped).")
```

Console Output:
```sh
172.16.20.40:22 is filtered (silently dropped).
172.16.20.40:23 is filtered (silently dropped).
172.16.20.40:80 is open.
172.16.20.40:443 is open.
172.16.20.40:449 is filtered (silently dropped).
```

>   For more information on TCP behavior and how this was created, visit: <a href="http://resources.infosecinstitute.com/port-scanning-using-scapy/" target="_blank" rel="noopener">Port Scanning Using Scapy</a>

#### ICMP Ping Sweep {#ping-sweep}

This script is an extension of our ICMP ping utility from the [Sending and Receiving](06-sending-and-receiving.html) example. We will use a network given with a CIDR mask to specify the hosts to run the ping scan on. Then, using a Python `for` loop we iterate through each address and try pinging. If the response times out or returns an ICMP error (such as unreachable or admin deny), we know that the host is not up or is blocking ICMP. Otherwise, if we receive a response we know that host is online. Check out the code here:

```python
#! /usr/bin/env python3

from ipaddress import IPv4Network
import random
from scapy.all import ICMP, IP, sr1, TCP

# Define IP range to ping
network = "192.168.40.0/24"

# make list of addresses out of network, set live host counter
addresses = IPv4Network(network)
live_count = 0

# Send ICMP ping request, wait for answer
for host in addresses:
    if (host in (addresses.network_address, addresses.broadcast_address)):
        # Skip network and broadcast addresses
        continue

    resp = sr1(
        IP(dst=str(host))/ICMP(),
        timeout=2,
        verbose=0,
    )

    if resp is None:
        print(f"{host} is down or not responding.")
    elif (
        int(resp.getlayer(ICMP).type)==3 and
        int(resp.getlayer(ICMP).code) in [1,2,3,9,10,13]
    ):
        print(f"{host} is blocking ICMP.")
    else:
        print(f"{host} is responding.")
        live_count += 1

print(f"{live_count}/{addresses.num_addresses} hosts are online.")
```

Console Output:
```sh
172.16.20.1 is responding.
WARNING: Mac address to reach destination not found. Using broadcast.
172.16.20.2 is down or not responding.
WARNING: Mac address to reach destination not found. Using broadcast.
172.16.20.3 is down or not responding.
172.16.20.4 is responding.
172.16.20.5 is responding.
172.16.20.6 is responding.
172.16.20.7 is responding.
WARNING: Mac address to reach destination not found. Using broadcast.
172.16.20.8 is down or not responding.
WARNING: Mac address to reach destination not found. Using broadcast.
172.16.20.9 is down or not responding.
172.16.20.10 is responding.
172.16.20.11 is responding.
... (truncated)
19/254 hosts are online.
```

NOTE: This could certainly be made much faster with threading since this is mostly IO bound (waiting for network responses), however that is outside the scope of this article.  
In this example, the `WARNING: Mac address to reach destination not found. Using broadcast.` is telling us that Scapy doesn't know the destination ARP address to send the packet to. This is only showing up because I am running this test on my locally connected network. If I were running this scan on a different network, Scapy would use the gateway MAC address for the L2 destination.

#### Combining the Two {#sweep-and-scan}

Those first two tools are cool, but you know what would be cooler? Combining them! With a new tool that combines those two features, we can scan a subnet for online hosts and then also run a TCP scan on the online hosts.

To get this new tool up and running, we can pretty much use the existing code with just a couple changes. We can get rid of the single `host` variable since we'll be using the network statment (this can define a single host using the /32 CIDR mask). Also, to keep the code clean and easy to understand we should move our TCP scan to it's own function so we can call that on any hosts that respond to the ICMP ping request. Here's the code and some example output:

```python
#! /usr/bin/env python3

import random
from ipaddress import IPv4Network
from typing import List

from scapy.all import ICMP, IP, sr1, TCP

# Define IP range to scan
network = "192.168.40.0/30"
# Define TCP port range
port_range = [22,23,80,443,449]

# make list of addresses out of network, set live host counter
addresses = IPv4Network(network)
live_count = 0

def port_scan(host: str, ports: List[int]):
    # Send SYN with random Src Port for each Dst port
    for dst_port in ports:
        src_port = random.randint(1025, 65534)
        resp = sr1(
            IP(dst=host)/TCP(sport=src_port,dport=dst_port,flags="S"),timeout=1,
            verbose=0,
        )
        if resp is None:
            print(f"{host}:{dst_port} is filtered (silently dropped).")

        elif(resp.haslayer(TCP)):
            if(resp.getlayer(TCP).flags == 0x12):
                send_rst = sr(
                    IP(dst=host)/TCP(sport=src_port,dport=dst_port,flags='R'),
                    timeout=1,
                    verbose=0,
                )
                print(f"{host}:{dst_port} is open.")

            elif (resp.getlayer(TCP).flags == 0x14):
                print(f"{host}:{dst_port} is closed.")

        elif(resp.haslayer(ICMP)):
            if(
                int(resp.getlayer(ICMP).type) == 3 and
                int(resp.getlayer(ICMP).code) in (1, 2, 3, 9, 10, 13)
            ):
                print(f"{host}:{dst_port} is filtered (silently dropped).")

# Send ICMP ping request, wait for answer
for host in addresses:
    if (host in (addresses.network_address, addresses.broadcast_address)):
        # Skip network and broadcast addresses
        continue

    resp = sr1(IP(dst=str(host))/ICMP(), timeout=2, verbose=0)

    if resp is None:
        print(f"{host} is down or not responding.")
    elif (
        int(resp.getlayer(ICMP).type)==3 and
        int(resp.getlayer(ICMP).code) in [1,2,3,9,10,13]
    ):
        print(f"{host} is blocking ICMP.")
    else:
        port_scan(str(host), port_range)
        live_count += 1

print(f"{live_count}/{addresses.num_addresses} hosts are online.")
```

Console Output:
```
172.16.20.1:22 is filtered (silently dropped).
172.16.20.1:23 is filtered (silently dropped).
172.16.20.1:80 is filtered (silently dropped).
172.16.20.1:443 is filtered (silently dropped).
172.16.20.1:449 is filtered (silently dropped).
172.16.20.2:22 is closed.
172.16.20.2:23 is closed.
172.16.20.2:80 is open.
172.16.20.2:443 is closed.
172.16.20.2:449 is closed.
172.16.20.3:22 is filtered (silently dropped).
172.16.20.3:23 is filtered (silently dropped).
172.16.20.3:80 is filtered (silently dropped).
172.16.20.3:443 is filtered (silently dropped).
172.16.20.3:449 is filtered (silently dropped).
172.16.20.4 is down or not responding.
172.16.20.5:22 is closed.
172.16.20.5:23 is closed.
172.16.20.5:80 is open.
172.16.20.5:443 is open.
172.16.20.5:449 is closed.
172.16.20.6:22 is closed.
172.16.20.6:23 is closed.
172.16.20.6:80 is open.
172.16.20.6:443 is closed.
172.16.20.6:449 is closed.
5/8 hosts are online.
```

That output is a little noisy for my taste, so if we remove some of the print statements like in this example <a title="10-scan-and-sweep-2.py" href="https://gist.github.com/thepacketgeek/7125755" target="_blank" rel="noopener">here</a>, we get the following output:

Console Output:
    ```
    172.16.20.1 is responding.
    172.16.20.2 is responding.
    172.16.20.2:80 is open.
    172.16.20.3 is responding.
    172.16.20.5 is responding.
    172.16.20.5:80 is open.
    172.16.20.5:443 is open.
    172.16.20.6 is responding.
    172.16.20.6:80 is open.
    Out of 8 hosts, 5 are online.
```

This is just the basics for building tools like this. The formatting can be customized to print out how you want and you can scan more ports if needed. Use these as starting points to build The Ultimate Network Tool and let me know what you create!