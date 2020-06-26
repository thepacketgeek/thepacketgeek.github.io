+++
title = "Scapy p.07"
description = "Monitoring ARP"
date = 2013-10-29
author = "Mat"
weight = 94

aliases = ["/scapy-p-07-monitoring-arp/"]

[taxonomies]
tags = ["scapy", "python"]
+++

#### Using Scapy in a Python Script

So far we've been working with Scapy in interactive mode. It's very powerful but there are times when it would be easier to work with a Python script instead. In order to use Scapy, we have to import the Scapy module like this:

```py
from scapy.all import *
```

This will import all Scapy functions, but if you know that you will only need a few of the functions, you can import them individually as well like this:

``` python
from scapy.all import sr1,IP,ICMP
```

<!-- more -->
The biggest different with running Scapy in a script is that the output may not be as verbose as interactive mode. If you're not getting all the output you need, make sure to try using the 

`print` command. Here's an example with our previous ping example:

```python
#! /usr/bin/env python3

from scapy.all import ICMP, sr1

print(sr1(IP(dst="4.2.2.1")/ICMP()).summary())
```

Console Output:
```sh
Begin emission:
..Finished to send 1 packets.
.*
Received 4 packets, got 1 answers, remaining 0 packets
IP / ICMP 4.2.2.1 > 172.16.20.40 echo-reply 0 / Padding
```

#### Using Scapy to monitor ARP traffic

Let's pretend that there is concern about someone potentially trying to use an ARP poisoning attack on our network. Using Scapy, we want to write a script that will listen to packets and print out all ARP requests and responses. We can do that very simply using the Scapy `sniff()` function and the `filter` argument like this:

```python
#! /usr/bin/env python3

from scapy.all import sniff

pkts = sniff(filter="arp", count=10)
print(pkts.summary())
```

Console Output:
```
Ether / ARP who has 172.16.20.10 says 172.16.20.40 / Padding
Ether / ARP who has 172.16.20.92 says 172.16.20.2 / Padding
Ether / ARP who has 172.16.20.2 says 172.16.20.92 / Padding
Ether / ARP who has 172.16.20.203 says 172.16.20.200 / Padding
Ether / ARP is at 00:11:22:29:a6:85 says 172.16.20.203
Ether / ARP who has 172.16.20.2 says 172.16.20.103 / Padding
Ether / ARP who has 172.16.20.103 says 172.16.20.2 / Padding
Ether / ARP who has 172.16.20.150 says 172.16.20.15 / Padding
Ether / ARP who has 172.16.20.2 says 172.16.20.19 / Padding
Ether / ARP who has 172.16.20.19 says 172.16.20.2 / Padding
```

This is a very simple script that gives us some good visibility to what's happening with ARP on our network. But what if we wanted to customize the output a little bit? Maybe we can get it in a format that would be easy to write to a file or send to some type of external monitoring server.

#### Using the prn Argument to Customize sniff() Output

I'm going to introduce the `prn` argument and show you how to do change what Scapy prints out for each packet:

```python
#! /usr/bin/env python3

from scapy.all import ARP, sniff

def arp_display(pkt):
    if pkt[ARP].op == 1:  # who-has (request)
        return f"Request: {pkt[ARP].psrc} is asking about {pkt[ARP].pdst}"
    if pkt[ARP].op == 2:  # is-at (response)
        return f"*Response: {pkt[ARP].hwsrc} has address {pkt[ARP].psrc}"

sniff(prn=arp_display, filter="arp", store=0, count=10)
```

Console Output:
```
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

Basically, what the `prn` argument lets us do is replace the default Scapy printout of the packet summary and run our own function to determine how Scapy prints out. That's really cool! It works by passing the packet object to the defined function, in this case `arp_display()`, each time a packet is sniffed that matches the specified filter.

We can do a lot more with that `prn` argument and passing more than just the packet object to the custom defined function using nested functions. That's outside the scope of this guide but feel free to read about it here: [Scapy Sniffing with Custom Actions](/scapy-sniffing-with-custom-actions-part-2/ "Scapy Sniffing with Custom Actions")
