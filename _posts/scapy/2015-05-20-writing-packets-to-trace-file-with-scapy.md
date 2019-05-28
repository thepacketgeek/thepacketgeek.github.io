---
title: Writing Packets to Trace File with Scapy
date: 2015-05-20T19:42:33-07:00
author: Mat
layout: post
permalink: /writing-packets-to-trace-file-with-scapy/
categories:
  - Scapy
---
This is just a quick follow-up post to accompany the previous <a href="//thepacketgeek.com/importing-packets-from-trace-files/" target="_blank" rel="noopener">Importing packets from trace files with scapy</a> post. So you've sniffed or generated some packets with scapy and it's time to write them to file to analyze and double-check your work. Here's a simple example of how to save those packets.

```python
localhost:~ packetgeek$ scapy
>>> packets = sniff(count=10)
>>> packets
<Sniffed: TCP:0 UDP:3 ICMP:0 Other:7>
>>> wrpcap('sniffed.pcap', packets)
```

Tada!  That's it. There's no options or special functions, you probably should do your packet processing before you write the packets to file.