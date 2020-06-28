+++
title = "Importing packets from trace files with Scapy"
date = 2014-09-25
author = "Mat"
in_search_index = true
aliases = ["/importing-packets-from-trace-files/"]
[taxonomies]
tags = ["scapy", "python"]
+++

Scapy is amazingly flexible when it comes to creating packets, but in some cases you may want to mangle or change packets that you've sniffed and saved in a trace file. Scapy currently supports .cap, .pcap, and .pcapng files.  Reading these files are possible through the `rdpcap()` function:

```
localhost:~ packetgeek$ scapy
>>> packets = rdpcap('IBGP_adjacency.cap')
>>> packets
<IBGP_adjacency.cap: TCP:17 UDP:0 ICMP:0 Other:0>
```
<!-- more -->
*Thanks to <a title="PacketLife.net" href="http://packetlife.net/captures/protocol/bgp/" target="_blank" rel="noopener">packetlife.net</a> for the iBGP capture found <a title="iBGP_adjacency.cap" href="https://www.cloudshark.org/captures/00249be4441f" target="_blank" rel="noopener">here.</a>


Then we can view, edit, change the packets like we could with any other packets that were sniffed or created with scapy.

```python
>>> packets.summary()
Ether / IP / TCP 4.4.4.4:11965 > 3.3.3.3:bgp S / Padding
Ether / IP / TCP 3.3.3.3:bgp > 4.4.4.4:11965 SA / Padding
Ether / IP / TCP 4.4.4.4:11965 > 3.3.3.3:bgp A / Padding
Ether / IP / TCP 4.4.4.4:11965 > 3.3.3.3:bgp PA / Raw
Ether / IP / TCP 3.3.3.3:bgp > 4.4.4.4:11965 PA / Raw
Ether / IP / TCP 4.4.4.4:11965 > 3.3.3.3:bgp PA / Raw
Ether / IP / TCP 3.3.3.3:bgp > 4.4.4.4:11965 PA / Raw
Ether / IP / TCP 4.4.4.4:11965 > 3.3.3.3:bgp PA / Raw
Ether / IP / TCP 3.3.3.3:bgp > 4.4.4.4:11965 PA / Raw
Ether / IP / TCP 4.4.4.4:11965 > 3.3.3.3:bgp PA / Raw
Ether / IP / TCP 4.4.4.4:11965 > 3.3.3.3:bgp PA / Raw
Ether / IP / TCP 3.3.3.3:bgp > 4.4.4.4:11965 A / Padding
Ether / IP / TCP 3.3.3.3:bgp > 4.4.4.4:11965 PA / Raw
Ether / IP / TCP 4.4.4.4:11965 > 3.3.3.3:bgp A / Padding
Ether / IP / TCP 4.4.4.4:11965 > 3.3.3.3:bgp PA / Raw
Ether / IP / TCP 3.3.3.3:bgp > 4.4.4.4:11965 PA / Raw
Ether / IP / TCP 4.4.4.4:11965 > 3.3.3.3:bgp A / Padding
>>>
>>> # Changing IP destination of BGP OPEN message
>>> packets[3][IP].dst='1.1.1.1'
>>> packets[3].summary()
'Ether / IP / TCP 4.4.4.4:11965 > 1.1.1.1:bgp PA / Raw'
```

## Reading .pcaps with a custom function

We can also use scapy's `sniff()` function to read packets from a .pcap file using the `offline` argument as show here:

```python
>>> packets = sniff(offline='IBGP_adjacency.cap')
>>>
>>> # packets is now the same list as in the previous example
>>> packets
<Sniffed: TCP:17 UDP:0 ICMP:0 Other:0>
```

This will allow us to use the prn() function to import the packets with custom functions, as covered in <a title="Scapy Sniffing with Custom Actions, Part 1" href="http://thepacketgeek.com/scapy-sniffing-with-custom-actions-part-1/" target="_blank" rel="noopener">this post</a>. Here we can count the packets as we import them from the .pcap file:

```python
>>> packetCount = 0
>>> def customAction(packet):
...    return f"{packet[0][1].src} ==> {packet[0][1].dst}"
...
>>> sniff(offline='IBGP_adjacency.cap', prn=customAction)
4.4.4.4 ==> 3.3.3.3
3.3.3.3 ==> 4.4.4.4
4.4.4.4 ==> 3.3.3.3
4.4.4.4 ==> 3.3.3.3
3.3.3.3 ==> 4.4.4.4
4.4.4.4 ==> 3.3.3.3
3.3.3.3 ==> 4.4.4.4
4.4.4.4 ==> 3.3.3.3
3.3.3.3 ==> 4.4.4.4
4.4.4.4 ==> 3.3.3.3
4.4.4.4 ==> 3.3.3.3
3.3.3.3 ==> 4.4.4.4
3.3.3.3 ==> 4.4.4.4
4.4.4.4 ==> 3.3.3.3
4.4.4.4 ==> 3.3.3.3
3.3.3.3 ==> 4.4.4.4
4.4.4.4 ==> 3.3.3.3
<Sniffed: TCP:17 UDP:0 ICMP:0 Other:0>
```

##  Manipulating packets during import

We can also make a more useful function during import that changes the packets as we import.  This example below just will clear the MAC addresses to keep physical device information anonymous, but with the power of python and a little imagination the possibilities are endless!

```python
>>> # create packet list
>>> packets = []
>>>
>>> # custom action function
>>> def customAction(packet):
...    packet[Ether].dst = "00:11:22:aa:bb:cc"
...    packet[Ether].src = "00:11:22:aa:bb:cc"
...
>>> for sniffed_packet in sniff(offline='IBGP_adjacency.cap', prn=customAction):
...     packets.append(sniffed_packet)
>>>
>>> # See edited MAC addresses in packet 'Ether' layer
>>> packets[0][Ether].summary()
'Ether / IP / TCP 4.4.4.4:11965 > 3.3.3.3:bgp S / Padding'
>>> packets[0][Ether].show()
>>> packets[0][Ether].show()
###[ Ethernet ]###
  dst= 00:11:22:aa:bb:cc
  src= 00:11:22:aa:bb:cc
  type= 0x800
...
... (truncated for brevity)
```

Check out [this post](writing-packets) in the series where we cover writing packets (sniffed, created, or mangled with Scapy) back to .cap/.pcap files!