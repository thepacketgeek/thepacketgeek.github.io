+++
title = "Intro to PyShark"
description = "for Programmatic Packet Analysis"
date = 2014-11-08
author = "Mat"
weight = 100
aliases = ["/intro-to-pyshark-for-programmatic-packet-analysis/"]
in_search_index = true
[taxonomies]
tags = ["pyshark", "python"]

+++

I can hardly believe it took me this long to find PyShark, but I am very glad I did! PyShark is a wrapper for the Wireshark CLI interface, tshark, so all of the Wireshark decoders are available to PyShark! It is so amazing that I started a new project just so I could use this amazing new tool: <a title="Cloud-Pcap" href="https://github.com/thepacketgeek/cloud-pcap" target="_blank">Cloud-Pcap</a>.

You can use PyShark to sniff from a interface or open a saved capture file, as the docs show on the <a title="PyShark" href="http://kiminewt.github.io/pyshark/" target="_blank">overview page here</a>:

```python
import pyshark

# Open saved trace file 
cap = pyshark.FileCapture('/tmp/mycapture.cap')

# Sniff from interface
capture = pyshark.LiveCapture(interface='eth0')
capture.sniff(timeout=10)
<LiveCapture (5 packets)>
```

<!-- more -->

Once a capture object is created, either from a LiveCapture or FileCapture method, several methods and attributes are available at both the capture and packet level.  The power of PyShark is the access to all of the packet decoders built into tshark.  I'm going to just give a sneak peek of some of the things you can do in this post and there will be a few accompanying posts that follow to go more in depth.

1. Getting packet summaries (similar to tshark capture output):

```sh
>>> for pkt in cap:
...:     print pkt
...:
2 0.512323 0.512323 fe80::f141:48a9:9a2c:73e5 ff02::c SSDP 208 M-SEARCH * HTTP/
3 1.331469 0.819146 fe80::159a:5c9f:529c:f1eb ff02::c SSDP 208 M-SEARCH * HTTP/
4 2.093188 0.761719 192.168.1.1 239.255.255.250 SSDP 395 NOTIFY * HTTP/1.  0x0000 (0)
5 2.096287 0.003099 192.168.1.1 239.255.255.250 SSDP 332 NOTIFY * HTTP/1.  0x0000 (0)
```

This will give access to attributes like packet number, relative and delta times, IP addresses, protocol, and a brief info line.

2. Drilling down into packet attributes by layer:

```sh
>>> pkt.   #(tab auto-complete)
pkt.captured_length     pkt.highest_layer       pkt.ip                  pkt.pretty_print        pkt.transport_layer
pkt.eth                 pkt.http                pkt.layers              pkt.sniff_time          pkt.udp
pkt.frame_info          pkt.interface_captured  pkt.length              pkt.sniff_timestamp
>>>
>>> pkt[pkt.highest_layer].    #(tab auto-complete)
pkt_app.                 pkt_app.get_field_value  pkt_app.raw_mode         pkt_app.request_version
pkt_app.DATA_LAYER       pkt_app.get_raw_value    pkt_app.request
pkt_app.chat             pkt_app.layer_name       pkt_app.request_method
pkt_app.get_field        pkt_app.pretty_print     pkt_app.request_uri
```

3. Iterating through the packets and applying a function to each:

```sh
>>> cap = pyshark.FileCapture('test.pcap', keep_packets=False)
>>> def print_highest_layer(pkt)
...: print pkt.highest_layer
>>> cap.apply_on_packets(print_highest_layer)
HTTP
HTTP
HTTP
HTTP
HTTP
... (truncated)
```

and this is just the sneak peak!!  Who knew that the getting the power of tshark & Wireshark in your python scripts and applications would be this easy!  The only caveat that I've found so far is the performance. I've thrown a lot of packets at PyShark and it can really slow down once you start running through captures of a couple thousand packets. Some things have been done to preserve memory that will be covered in the following posts.

I certainly hope you're as excited as I am at this point. There's plenty more to come, so check back soon!