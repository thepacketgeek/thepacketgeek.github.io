+++
title = "Using the Packet Object"
date = 2014-11-12
author = "Mat"
weight = 70
in_search_index = true
aliases = ["/pyshark-using-the-packet-object/"]
[taxonomies]
tags = ["pyshark", "python"]
+++
So far in this series we've done a lot with capturing packets and working with the capture object, but finally we're going to get to the fun part and finally start playing with some PACKETS!!!!

When we have captured packets in a capture object, they are stored as a list of packet objects.  These packet objects will have methods and attributes that give us access to the header and payload info of each packet.  As stated in a previous post we have control for how much info about the packets we store in each packet option through the `only_summaries` argument in the LiveCapture and ReadCapture modules.
<!-- more -->

### Packet Summary Attributes

Setting `only_summaries` to True during capture will give us a fixed set of attributes, regardless of the protocols present in the packet. The most useful attributes available are:

```python
>>> cap = pyshark.FileCapture('test.pcap', only_summaries=True)
>>>
>>> dir(cap[0])
['delta', 'destination', 'info', 'ip id', 'length', 'no', 'protocol', 'source', 'stream', 'summary_line', 'time', 'window']
```

  * **delta**: Delta (difference) time between the current packet and the previous captured packet
  * **destination**: The Layer 3 (IP, IPv6) destination address
  * **info**: A brief application layer summary (e.g. &#8216;HTTP GET /resource_folder/page.html')
  * **ip id**: IP Identification field used for uniquely identifying packets from a host
  * **length**: Length of the packet in bytes
  * **no**: Index number of the packet in the list
  * **protocol**: The highest layer protocol recognized in the packet
  * **source**:  Layer 3 (IP, IPV6) source address
  * **stream**: Index of the TCP stream this packet is a part of (TCP packets only)
  * **summary_line**: All the summary attributes in one tab-delimited string
  * **time**: Absolute time between the current packet and the first packet
  * **window**: The TCP window size (TCP packets only)

There's a lot that you can do with just these items; printing out packet summaries is just the beginning! Great visual charts can be made to illustrate IP conversations, bandwidth usage, protocol breakdowns, and application performance measurements (round-trip-times in the same TCP stream). Wow, that's a lot of useful analysis.. but wait, there's more?!?

### Full Packet Attributes

If you're wanting to get more than just the summary info out of the capture packets then you're in the right place. Using the dissectors available in Wireshark and tshark, PyShark is able to break out all packet details by layer. For example, let's dig into this DNS packet first by looking at the attributes of the parent packet object:

```py
>>> cap = pyshark.LiveCapture(interface='en0', bpf_filter='udp port 53')
>>> cap.sniff(packet_count=50)

>>> dns_1 = cap[0]
>>> dns_2 = cap[1]
>>> dns_1.      #(tab auto-complete)
dns_1.captured_length     dns_1.highest_layer       dns_1.length              dns_1.transport_layer
dns_1.dns                 dns_1.interface_captured  dns_1.pretty_print        dns_1.udp
dns_1.eth                 dns_1.ip                  dns_1.sniff_time
dns_1.frame_info          dns_1.layers              dns_1.sniff_timestamp
```

There are several generic packet info attributes for length, frame_info, and time, and a `pretty_print()` method for displaying the packet in a very readable format (similar to Wireshark's packet detail view). If you look close though you'll see attributes that represent absolute layers (eth and ip) as well as attributes that change based on the protocols present in each packet (transport\_layer, highest\_layer, dns).  If you're looking for  specific traffic, these attributes make it very easy to locate the packets containing interesting information, like this script that will print out all DNS query and response names:

```py
import pyshark

cap = pyshark.LiveCapture(interface='en0', bpf_filter='udp port 53')

cap.sniff(packet_count=10)

def print_dns_info(pkt):
    if pkt.dns.qry_name:
        print 'DNS Request from %s: %s' % (pkt.ip.src, pkt.dns.qry_name)
    elif pkt.dns.resp_name:
        print 'DNS Response from %s: %s' % (pkt.ip.src, pkt.dns.resp_name)

cap.apply_on_packets(print_dns_info, timeout=100)
```

Which gives the output:

```sh
DNS Request from 10.10.10.40: apple.com
DNS Request from 10.10.10.1: apple.com
DNS Request from 10.10.10.40: ipv6.icanhazip.com
DNS Request from 10.10.10.1: ipv6.icanhazip.com
DNS Request from 10.10.10.40: ipv4.icanhazip.com
DNS Request from 10.10.10.1: ipv4.icanhazip.com
```

###  Dynamic Layer References

Using the dynamic layer attributes I mentioned earlier gives us some flexibility when analyzing packets. If you try to access the `pkt.dns.qry_resp` attribute of every packet, you will receive an `AttributeError` if the packet doesn't have DNS info. This also applies to transport layer, since each packet will have either a TCP or UDP layer. We can print out the source and destination addresses and ports (for IP conversation mapping) and use a `try/except` loop to protect against the `AttributeError` if the packet is neither TCP nor UDP:

```py
import pyshark

cap = pyshark.FileCapture('test.pcap')

def print_conversation_header(pkt):
    try:
        protocol =  pkt.transport_layer
        src_addr = pkt.ip.src
        src_port = pkt[pkt.transport_layer].srcport
        dst_addr = pkt.ip.dst
        dst_port = pkt[pkt.transport_layer].dstport
        print '%s  %s:%s --> %s:%s' % (protocol, src_addr, src_port, dst_addr, dst_port)
    except AttributeError as e:
        #ignore packets that aren't TCP/UDP or IPv4
        pass

cap.apply_on_packets(print_conversation_header, timeout=100)
```

Which gives the output:

```sh
UDP  10.10.10.12:51554 --> 239.255.255.250:1900
UDP  10.10.10.12:51554 --> 239.255.255.250:1900
UDP  10.10.10.15:58803 --> 8.8.8.8:53
UDP  8.8.8.8:53 --> 10.10.10.15:58803
TCP  10.10.10.15:58632 --> 192.168.20.197:80
TCP  192.168.20.197:80 --> 10.10.10.15:58632
TCP  10.10.10.15:58632 --> 192.168.20.197:80
```

###  Endless Possibilities

As you can see from just these few examples is that PyShark gives access to all packet details with ease. Conditional statements could be used to create dynamic logic for categorizing and working with many different protocols, or you can search for certain types of traffic that have a specific attribute (make sure to protect for `AttributeErrors`).

I hope you've enjoyed these last few posts. I'd love to hear about projects that you come up with using PyShark, so feel free to share! If you'd like to see one example of PyShark out in the wild, check out my project <a title="GitHub: Cloud-Pcap" href="https://github.com/thepacketgeek/cloud-pcap" target="_blank">Cloud-Pcap</a>.