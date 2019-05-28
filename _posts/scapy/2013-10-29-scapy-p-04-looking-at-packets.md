---
title: Scapy p.04 – Looking at Packets
date: 2013-10-29T16:59:56-07:00
author: Mat
layout: post
permalink: /scapy-p-04-looking-at-packets/
wp_last_modified_info:
  - February 14, 2019 @ 1:06 pm
categories:
  - Scapy
---
#### Packets, Layers, and Fields. Oh My!

Scapy uses Python dictionaries as the data structure for packets. Each packet is a collection of nested dictionaries with each layer being a child dictionary of the previous layer, built from the lowest layer up. Visualizing the nested packet layers would look something like this:  
<img class="aligncenter size-full" src="{{ site.url }}/static/img/scapy-packet-layers.png" alt="pkt-layers" width="494" height="186" sizes="(max-width: 494px) 100vw, 494px" />

Each field (such as the Ethernet `dst' value or ICMP `type' value) is a key:value pair in the appropriate layer. These fields (and nested layers) are all mutable so we can reassign them in place using the assignment operator. Scapy has packet methods for viewing the layers and fields that I will introduce next.

#### Packet summary() and show() Methods

Now let's go back to our `pkt` and have some fun with it using Scapy's Interactive mode. We already know that using the `summary()` method will give us a quick look at the packet's layers:

```python
>>> pkt[0].summary()
'Ether / IP / ICMP 172.16.20.10 > 4.2.2.1 echo-request 0 / Raw'
```

But what if we want to see more of the packet contents? That's what the 

`show()` method is for:

```python
>>> pkt[0].show()
###[ Ethernet ]###
  dst= 00:24:97:2e:d6:c0
  src= 00:00:16:aa:bb:cc
  type= 0x800
###[ IP ]###
     version= 4L
     ihl= 5L
     tos= 0x0
     len= 84
     id= 57299
     flags= 
     frag= 0L
     ttl= 64
     proto= icmp
     chksum= 0x0
     src= 172.16.20.10
     dst= 4.2.2.1
     \options\
###[ ICMP ]###
        type= echo-request
        code= 0
        chksum= 0xd8af
        id= 0x9057
        seq= 0x0
###[ Raw ]###
```

Very cool, that's some good info. If you're familiar with Python you have probably noticed the list index, `[0]`, after the `pkt` variable name. Remember that our sniff only returned a single packet, but if we increase the `count` argument value, we will get back an list with multiple packets:

```python
>>> pkts = sniff(count=10)
>>> pkts
<Sniffed: TCP:0 UDP:0 ICMP:10 Other:0>
```

<p class="caption">
  Getting the value of the list returns a quick glance at what type of packets were sniffed.
</p>

```python
>>> pkts.summary()
Ether / IP / ICMP 172.16.20.10 > 4.2.2.1 echo-request 0 / Raw
Ether / IP / ICMP 4.2.2.1 > 172.16.20.10 echo-reply 0 / Raw
Ether / IP / ICMP 172.16.20.10 > 4.2.2.1 echo-request 0 / Raw
Ether / IP / ICMP 4.2.2.1 > 172.16.20.10 echo-reply 0 / Raw
Ether / IP / ICMP 172.16.20.10 > 4.2.2.1 echo-request 0 / Raw
Ether / IP / ICMP 4.2.2.1 > 172.16.20.10 echo-reply 0 / Raw
Ether / IP / ICMP 172.16.20.10 > 4.2.2.1 echo-request 0 / Raw
Ether / IP / ICMP 4.2.2.1 > 172.16.20.10 echo-reply 0 / Raw
Ether / IP / ICMP 172.16.20.10 > 4.2.2.1 echo-request 0 / Raw
Ether / IP / ICMP 4.2.2.1 > 172.16.20.10 echo-reply 0 / Raw
```

And we can show the summary or packet contents of any single packet by using the list index with that packet value. So, let's look at the contents of the 4th packet (Remember, list indexes start counting at 0):

```python
>>> pkts[3]
<Ether  dst=00:00:16:aa:bb:cc src=00:24:97:2e:d6:c0 type=0x800 |<IP  version=4L ihl=5L tos=0x20 len=84 id=47340 flags= frag=0L ttl=57 proto=icmp chksum=0x3826 src=4.2.2.1 dst=172.16.20.10 options=[] |<ICMP  type=echo-reply code=0 chksum=0xcfbf id=0x3060 seq=0x1 |<Raw |>>>>
```

<p class="caption">
  Getting the value of a single packet returns a quick glance of the contents of that packet.
</p>

The `show()` method will give us a cleaner print out:

```python
>>> pkts[3].show()
###[ Ethernet ]###
  dst= 00:00:16:aa:bb:cc
  src= 00:24:97:2e:d6:c0
  type= 0x800
###[ IP ]###
     version= 4L
     ihl= 5L
     tos= 0x20
     len= 84
     id= 47340
     flags= 
     frag= 0L
     ttl= 57
     proto= icmp
     chksum= 0x3826
     src= 4.2.2.1
     dst= 172.16.20.10
     \options\
###[ ICMP ]###
        type= echo-reply
        code= 0
        chksum= 0xcfbf
        id= 0x3060
        seq= 0x1
###[ Raw ]###
```

#### Digging into Packets by Layer

Scapy builds and dissects packets by the layers contained in each packet, and then by the fields in each layer. Each layer is nested inside the parent layer as can be seen with the nesting of the < and > brackets:

```python
>>> pkts[4]
<Ether  dst=00:24:97:2e:d6:c0 src=00:00:16:aa:bb:cc type=0x800 |<IP  version=4L ihl=5L tos=0x0 len=84 id=17811 flags= frag=0L ttl=64 proto=icmp chksum=0x0 src=192.168.201.203 dst=4.2.2.1 options=[] |<ICMP  type=echo-request code=0 chksum=0xc378 id=0x3060 seq=0x2 |<Raw |>>>>
```

You can also dig into a specific layer using an list index. If we wanted to get to the `ICMP` layer of `pkts[3]`, we could do that using the layer name or index number:

```python
>>> pkts[3][ICMP].summary()
'ICMP 4.2.2.1 > 192.168.201.203 echo-reply 0 / Raw'
>>> pkts[3][2].summary()
'ICMP 4.2.2.1 > 192.168.201.203 echo-reply 0 / Raw'
```

Since the first index chooses the packet out of the `pkts` list, the second index chooses the layer for that specific packet. Looking at the summary of this packet from an earlier example, we know that the `ICMP` layer is the 3rd layer.

#### Packet .command() Method

If you're wanting to see a reference of how a packet that's been sniffed or received might look to create, Scapy has a packet method for you! Using the `.command()` packet method will return a string of the command necessary to recreate that packet, like this:

```python
>>> pkts[2].command()
'Ether(src=\'00:11:22:aa:bb:cc\', dst=\'c0:c1:c0:b7:ce:63\', type=2048)/IP(frag=0L, src=\'172.16.20.10\', proto=1, tos=0, dst=\'4.2.2.1\', chksum=51457, len=84, options=[], version=4L, flags=0L, ihl=5L, ttl=64, id=59755)/ICMP(gw=None, code=0, ts_ori=None, addr_mask=None, seq=3, ptr=None, unused=None, ts_rx=None, chksum=50424, reserved=None, ts_tx=None, type=8, id=59999)/Raw(load=\'Rk\\xe8\\x02\\x00\\x0c#\\\'\\x08\\t\\n\\x0b\\x0c\\r\\x0e\\x0f\\x10\\x11\\x12\\x13\\x14\\x15\\x16\\x17\\x18\\x19\\x1a\\x1b\\x1c\\x1d\\x1e\\x1f !"#$%&\\\'()*+,-./01234567\')'
```

#### Digging into Layers by Field

Within each layer, Scapy parses out the value of each field if it has support for the layer's protocol. Depending on the type of field, Scapy may replace the value with a more friendly text value for the summary views, but not in the values returned for an individual field. Here are some examples:

```python
>>> pkts[3]
<Ether  dst=00:00:16:aa:bb:cc src=00:24:97:2e:d6:c0 type=0x800 |\
<IP  version=4L ihl=5L tos=0x20 len=84 id=47340 flags= frag=0L ttl=57 proto=icmp chksum=0x3826 src=4.2.2.1 dst=192.168.201.203 options=[] |\
<ICMP  type=echo-reply code=0 chksum=0xcfbf id=0x3060 seq=0x1 |<Raw |>>>>
>>> pkts[3][Ether].src
'00:24:97:2e:d6:c0'
>>> pkts[3][IP].ttl
57
>>> pkts[3][IP].proto
1
>>> pkts[3][ICMP].type
0
```

#### Using Python control statements with Scapy

The awesome thing about Scapy being a module of Python is that we can use the power of Python to do stuff with our packets. Here's a tip of the iceberg example using a Python `for` statement along with some new Scapy packet methods:

```python
>>> for packet in pkts:
...     if (packet.haslayer(ICMP)):
...         print(f"ICMP code: {packet.getlayer(ICMP).code}")
...
ICMP code: 0
ICMP code: 0
ICMP code: 0
ICMP code: 0
ICMP code: 0
ICMP code: 0
ICMP code: 0
ICMP code: 0
ICMP code: 0
ICMP code: 0
```

As you can guess, the `haslayer()` and `getlayer()` methods will test for the existence of a layer and return the layer (and any nested layers) respectively. This is just a very basic use of Python statements with Scapy and we'll see a lot more when it comes to packet generation and custom actions.