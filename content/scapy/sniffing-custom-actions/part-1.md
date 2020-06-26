+++
title = "Scapy Sniffing with Custom Actions"
description = "Part 1"
date = 2013-10-01
author = "Mat"
aliases = [
    "/scapy-sniffing-with-custom-actions-part-1/"
]

[taxonomies]
tags = ["scapy", "python"]
+++

Scapy has a `sniff` function that is great for getting packets off the wire, but there's much more to show off how great this function really is! `sniff` has  an argument `prn` that allows you to pass a function that executes with each packet sniffed. The intended purpose of this function is to control how the packet prints out in the console allowing you to replace the default packet printing display with a format of your choice.

The `prn` argument is defined as:

> prn: function to apply to each packet. If something is returned, it is displayed. For instance you can use prn = lambda x: x.summary().

<!-- more -->

In order for your program/script to format and return the packet info as you wish, the `sniff` function passes the packet object as the one and only argument into the function you specify in the sniff's `prn` argument. This gives us the option to do some fun stuff (not just formatting) with each packet sniffed 🙂

For example, we can now perform custom actions with each sniffed packet. This can be anything from incrementing a packet count somewhere in the program, to doing some advanced packet parsing or manipulation, or even shipping that packet off into some sort of storage (.pcap appending or API POSTing anyone??).

## Here's a simple example for keeping track of the number of packets sniffed

This script keeps a Counter with an A/Z pair of IP addresses, displays the total packet count with each packet `print()`, and then prints out the conversation counts at the end.

```python
#! /usr/bin/env python3

from collections import Counter
from scapy.all import sniff

## Create a Packet Counter
packet_counts = Counter()

## Define our Custom Action function
def custom_action(packet):
    # Create tuple of Src/Dst in sorted order
    key = tuple(sorted([packet[0][1].src, packet[0][1].dst]))
    packet_counts.update([key])
    return f"Packet #{sum(packet_counts.values())}: {packet[0][1].src} ==> {packet[0][1].dst}"

## Setup sniff, filtering for IP traffic
sniff(filter="ip", prn=custom_action, count=10)

## Print out packet count per A <--> Z address pair
print("\n".join(f"{f'{key[0]} <--> {key[1]}'}: {count}" for key, count in packet_counts.items()))
```

Console Output:
```sh
Packet #1: 172.16.200.88 ==> 255.255.255.255
Packet #2: 172.16.200.88 ==> 172.16.98.255
Packet #3: 172.16.200.88 ==> 255.255.255.255
Packet #4: 172.16.200.88 ==> 255.255.255.255
Packet #5: 172.16.72.72 ==> 172.16.98.203
Packet #6: 172.16.98.203 ==> 172.16.72.72
Packet #7: 172.16.98.203 ==> 172.16.72.72
Packet #8: 172.16.72.72 ==> 172.16.98.203
Packet #9: 172.16.72.72 ==> 172.16.98.203
Packet #10: 172.16.98.203 ==> 172.16.72.72
172.16.200.88 <--> 255.255.255.255: 3
172.16.200.88 <--> 172.16.98.255: 1
172.16.72.72 <--> 172.16.98.203: 6
```

## Custom Formatted ARP Monitor

Here I use the same `prn` function and some conditional statements to very clearly tell me what ARP traffic my computer is seeing.

```python
#! /usr/bin/env python3

from collections import Counter
from scapy.all import ARP, sniff

def arp_display(pkt):
    if pkt[ARP].op == 1: #who-has (request)
        return f"Request: {pkt[ARP].psrc} is asking about {pkt[ARP].pdst}"
    if pkt[ARP].op == 2: #is-at (response)
        return f"*Response: {pkt[ARP].hwsrc} has address {pkt[ARP].psrc}"

sniff(prn=arp_display, filter="arp", store=0, count=10)
```

Console Output:
```sh
Request: 172.16.20.168 is asking about 172.16.20.85
Request: 172.16.20.168 is asking about 172.16.20.229
Request: 172.16.20.2 is asking about 172.16.20.203
*Response: aa:bb:cc:29:a6:85 has address 172.16.20.203
Request: 172.16.20.200 is asking about 172.16.20.82
Request: 172.16.20.51 is asking about 172.16.20.254
Request: 172.16.20.15 is asking about 172.16.20.58
Request: 172.16.20.15 is asking about 172.16.20.58
Request: 172.16.20.200 is asking about 172.16.20.44
*Response: dd:ee:ff:a2:02:bf has address 172.16.20.44
```


### An important thing to keep in mind when using the `prn` argument

In the case of the example above, you are passing the `custom_action` function **into** the `sniff` function. If you used `sniff(prn=custom_action())` instead, you would be passing the function's returned value to the sniff function. This will generate the returned text before the function has a packet to parse and will not give you the results you want.

If you want to pass parameters into the `custom_action` function for additional control or the ability to modularize out the `customAction` function, you will have to use a nested function. I cover how to do that in the next article.