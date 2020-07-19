+++
title = "Writing Packets to Trace File with Scapy"
date = 2015-05-20
author = "Mat"
in_search_index = true
aliases = ["/writing-packets-to-trace-file-with-scapy/"]
[taxonomies]
tags = ["scapy", "python"]

+++

This is a follow-up post to accompany the previous [importing packets from trace files with scapy](@/scapy/importing-pcaps.md) post. So you've sniffed or generated some packets with scapy and it's time to write them to file to analyze and double-check your work. Here's a simple example of how to save those packets.

```python
localhost:~ packetgeek$ scapy
>>> packets = sniff(count=10)
>>> packets
<Sniffed: TCP:0 UDP:3 ICMP:0 Other:7>
>>> wrpcap('sniffed.pcap', packets)
```
<!-- more -->
Tada!  That's it. There's no options or special functions, you probably should do your packet processing before you write the packets to file.