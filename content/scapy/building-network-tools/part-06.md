+++
title = "Scapy p.06"
description = "Sending and Receiving with Scapy"
date = 2013-10-29
author = "Mat"
weight = 95

aliases = ["/scapy-p-06-sending-and-receiving-with-scapy/"]
[taxonomies]
tags = ["scapy", "python"]
+++

We've sniffed some packets, dig down into packet layers and fields, and even sent some packets. Great job! It's time to step up our game with Scapy and start really using some of the power Scapy contains. Please Note: this next example is for education and example only. Please be responsible on your network, especially at work!

#### Scapy Send/Receive Function

Let's get familiar with the `sr()`, `sr1()`, `srp()`, and `srp1()` functions. Just like the `send()`, function, the `p` at the end of the function name means that we're sending at L2 instead of L3. The functions with a `1` in them mean that Scapy will send the specified packet and end after receiving 1 answer/response instead of continuing to listen for answers/responses. I'll reference both functions as `sr()`, but the examples will use the correct function.

#### <!--more-->Sending an ICMP Echo Request (ping)

The `sr()` function is used to send a packet or group of packets when you expect a response back. We'll be sending an ICMP Echo Request (ping) since we can expect some sort of a response back from that. First let's use the `sniff()` function to figure out what an ICMP Echo Request looks like in Scapy:

```python
> p = sniff(count=10,filter="icmp and ip host 4.2.2.1")
> p
<Sniffed: TCP:0 UDP:0 ICMP:10 Other:0>
> p[0]
<Ether  dst=00:07:7d:6d:b4:9e src=b8:f6:b1:11:65:35 type=0x800 |<IP  version=4L ihl=5L tos=0x0 len=84 id=14488 flags= frag=0L ttl=64 proto=icmp chksum=0x7bd6 src=172.16.20.40 dst=4.2.2.1 options=[] |<ICMP  type=echo-request code=0 chksum=0xaba6 id=0x55d3 seq=0x0 |<Raw |>>
```

In the previous ARP example we changed the dst and src MAC address, but since we're expecting a response back from another network device we'll have to leave it up to Scapy to fill those in when it sends the packets. Since we're building a L3 packet, we can actually leave off the Ether layer since Scapy will handle the generation of that. So let's start building the `IP` and `ICMP` layers. To see the available fields for each layer, and what the default values will be if we don't specify, use the `ls('layer')` command:

```python
>>> ls(IP)
version    : BitField             = (4)
ihl        : BitField             = (None)
tos        : XByteField           = (0)
len        : ShortField           = (None)
id         : ShortField           = (1)
flags      : FlagsField           = (0)
frag       : BitField             = (0)
ttl        : ByteField            = (64)
proto      : ByteEnumField        = (0)
chksum     : XShortField          = (None)
src        : Emph                 = (None)
dst        : Emph                 = ('127.0.0.1')
options    : PacketListField      = ([])
>>> ls(ICMP)
type       : ByteEnumField        = (8)
code       : MultiEnumField       = (0)
chksum     : XShortField          = (None)
id         : ConditionalField     = (0)
seq        : ConditionalField     = (0)
ts_ori     : ConditionalField     = (13940582)
ts_rx      : ConditionalField     = (13940582)
ts_tx      : ConditionalField     = (13940582)
gw         : ConditionalField     = ('0.0.0.0')
ptr        : ConditionalField     = (0)
reserved   : ConditionalField     = (0)
addr_mask  : ConditionalField     = ('0.0.0.0')
unused     : ConditionalField     = (0)
```

Most of those default values are fine, and src addresses will be filled out automatically by Scapy when it sends the packets. We can spoof those if desired, but again, since we're expecting a response we need to leave those alone. We'll be sending a L3 packet and we only expect one response, so we'll build and send our ICMP packet using the `sr1()` function:

```python
>>> pingr = IP(dst="192.168.200.254")/ICMP()
>>> sr1(pingr)
Begin emission:
..Finished to send 1 packets.
.*
Received 84 packets, got 1 answers, remaining 0 packets
<IP  version=4L ihl=5L tos=0x0 len=28 id=1 flags= frag=0L ttl=255 proto=icmp chksum=0xa7c4 src=4.2.2.1 dst=172.16.20.40 options=[] |<ICMP  type=echo-reply code=0 chksum=0xffff id=0x0 seq=0x0 |<Padding |>>>
```

>  Scapy prints out the response packet

The `Received 84 packets` is referring to the number of non-response packets Scapy sniffed while waiting for the response. It's not anything to be alarmed about, but just note that on a busy host you might see a big number of packets there. We can also define the ICMP packet directly in the `sr1()` function like this:

```python
>>> sr1(IP(dst="192.168.200.254")/ICMP())
Begin emission:
..Finished to send 1 packets.
.*
Received 97 packets, got 1 answers, remaining 0 packets
<IP  version=4L ihl=5L tos=0x0 len=28 id=1 flags= frag=0L ttl=255 proto=icmp chksum=0xa7c4 src=4.2.2.1 dst=172.16.20.40 options=[] |<ICMP  type=echo-reply code=0 chksum=0xffff id=0x0 seq=0x0 |<Padding |>>>
```

We can save the response packet into a variable just like we do when creating a packet:

```python
>>> resp = sr1(pingr)
Begin emission:
..Finished to send 1 packets.
.*
Received 147 packets, got 1 answers, remaining 0 packets
>>> resp[0].summary()
'IP / ICMP 4.2.2.1 > 172.16.20.40 echo-reply 0 / Padding'
```

> If we're saving the response, Scapy won't print it out by default

Two other Scapy functions related to sending and receiving packets are the `srloop()` and `srploop()`. The `srloop()` will send the L3 packet and continue to resend the packet after each response is received. The `srploop()` does the same thing except for... you guessed it, L2 packets! This let's us simulate the `ping` command, and with the `count` argument, we can also define the number of times to loop:

```python
>>> resp = srloop(pingr, count=5)
RECV 1: IP / ICMP 4.2.2.1 > 172.16.20.10 echo-reply 0 / Padding
RECV 1: IP / ICMP 4.2.2.1 > 172.16.20.10 echo-reply 0 / Padding
RECV 1: IP / ICMP 4.2.2.1 > 172.16.20.10 echo-reply 0 / Padding
RECV 1: IP / ICMP 4.2.2.1 > 172.16.20.10 echo-reply 0 / Padding
RECV 1: IP / ICMP 4.2.2.1 > 172.16.20.10 echo-reply 0 / Padding

Sent 5 packets, received 5 packets. 100.0% hits.
>>> resp
(<Results: TCP:0 UDP:0 ICMP:5 Other:0>, <PacketList: TCP:0 UDP:0 ICMP:0 Other:0>)
```

> Saving the responses puts our packets into an array

As you can see, our Scapy skills are building and you might already have some ideas about how you can use these functions in your own network tools. In the next article, we'll see how you can build an ARP monitor to keep an ear on the network for possible spoofed ARP replies.