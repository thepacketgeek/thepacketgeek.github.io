---
id: 349
title: 'PyShark &#8211; FileCapture and LiveCapture modules'
date: 2014-11-10T12:04:33-07:00
author: Mat
layout: post
guid: http://thepacketgeek.com/?p=349
permalink: /pyshark-filecapture-and-livecapture-modules/
categories:
  - Coding
tags:
  - PyShark
  - Python
series:
  - Intro to PyShark
---
<div class="seriesmeta">
  This entry is part 2 of 4 in the series <a href="https://thepacketgeek.com/series/intro-to-pyshark/" class="series-20" title="Intro to PyShark">Intro to PyShark</a>
</div>

The two typical ways to start analyzing packets are via PyShark&#8217;s FileCapture and LiveCapture modules. The first will import packets from a saved capture file, and the latter will sniff from a network interface on the local machine. Running these modules will return a capture object which I will cover in depth in the next post. For now, let&#8217;s see what we can do with these two modules.

Both modules offer similar parameters that affect packets returned in the capture object. These definitions are taken directly out of the docstrings for these modules:

  * **interface**:** **[LiveCapture only]** **Name of the interface to sniff on. If not given, takes the first available.**  
** 
  * **bpf_filter**: [LiveCapture only] A BPF (tcpdump) filter to apply on the cap before reading.
  * **input_file**: [FileCapture only] File path of the capture (PCAP, PCAPNG)
  * **keep_packets**: Whether to keep packets after reading them via next(). Used to conserve memory when reading large caps.
  * **display_filter:** A display (wireshark) filter to apply on the cap before reading it.
  * **only_summaries**: Only produce packet summaries, much faster but includes very little information.
  * **decryption_key**: Optional key used to encrypt and decrypt captured traffic.
  * **encryption_type**: Standard of encryption used in captured traffic (must be either &#8216;WEP&#8217;, &#8216;WPA-PWD&#8217;, or &#8216;WPA-PWK&#8217;. Defaults to WPA-PWK).

### <!--more-->Only_Summaries

Using `only_summaries` will return packets in the capture object with just the summary info of each packet (similar to the default output of tshark):

<pre class="lang:default decode:true">&gt;&gt;&gt; cap = pyshark.FileCapture('test.pcap', only_summaries=True)
&gt;&gt;&gt; print cap[0]
2 0.512323 0.512323 fe80::f141:48a9:9a2c:73e5 ff02::c SSDP 208 M-SEARCH * HTTP/
</pre>

This option makes the capture file reading much faster, although each packet will only have the attributes shown below available. This info can be plenty if you&#8217;re just wanting to get the IP addresses to build a conversation list in the sniff, or maybe some bandwidth statistics with the time and packet lengths:

<pre class="lang:default decode:true">&gt;&gt;&gt; pkt.     #(tab auto-complete)
pkt.delta         pkt.info          pkt.no            pkt.stream        pkt.window
pkt.destination   pkt.ip id         pkt.protocol      pkt.summary_line
pkt.host          pkt.length        pkt.source        pkt.time</pre>

### **Keep_Packets**

PyShark only reads packets into memory when it&#8217;s about to do something with the packets. As you work through the packets, PyShark appends each packet to a list attribute of the capture object named `_packet`. When working with a large amount of packets this list can take up a lot of memory so PyShark gives us the option to only keep one packet in memory at a time. If `keep_packets` is set to False (default is True), PyShark will read in a packet and then flush it from memory when it moves on to read in the next packet. I have found that this speeds up the processing time of packet iteration a bit, and every second helps!

### Display\_Filter and BPF\_Filter

The filters available in these modules can be helpful in keeping your application focused on the traffic you&#8217;re wanting to analyze. Similar to Wireshark or tshark sniffing, a BPF filter can be used to specify interesting traffic that makes it into the returned capture object. BPF filters don&#8217;t offer as much flexibility as Wireshark&#8217;s display filters, but you&#8217;d be surprised how creative you can be with the available keywords and offset filters. For help with BPF filters used in capturing packets, check out <a title="Wireshark Capture Filters" href="http://wiki.wireshark.org/CaptureFilters" target="_blank">Wireshark&#8217;s guide here</a>. Here&#8217;s an example of using a BPF filter when sniffing to target HTTP traffic:

<pre class="lang:default decode:true ">&gt;&gt;&gt; cap = pyshark.LiveCapture(interface='en0', bpf_filter='ip and tcp port 80')
&gt;&gt;&gt; cap.sniff(timeout=5)
&gt;&gt;&gt; cap
   &lt;LiveCapture (21 packets)&gt;
&gt;&gt;&gt; print cap[5].highest_layer
HTTP
</pre>

When reading in a saved capture file, you can use the `display_filter` option to harness Wireshark&#8217;s amazing dissectors to limit the packets returned. Here&#8217;s the first few packets in my test.pcap file without a filter:

<pre class="lang:default decode:true">&gt;&gt;&gt; cap = pyshark.FileCapture('test.pcap')
&gt;&gt;&gt; for pkt in cap:
...:    print pkt.highest_layer
...:
HTTP
HTTP
HTTP
TCP
HTTP
... (truncated)</pre>

And with a display filter for DNS traffic only:

<pre class="lang:default decode:true">&gt;&gt;&gt; cap = pyshark.FileCapture('test.pcap', display_filter="dns")
&gt;&gt;&gt; for pkt in cap:
...:    print pkt.highest_layer
...:
DNS
DNS
DNS
DNS
DNS
... (truncated)</pre>

###  Caveats with LiveCapture

I discovered a strange behavior when trying to iterate through a LiveCapture returned capture object. It appears that when you try to iterate through the list, it starts the sniff over again and iterates in real time (as packets are received on the interface). There&#8217;s no way (that I&#8217;ve found yet) to store the packets, the LiveCapture is meant to process packets in real time only.

&#8212;

There are some powerful options for opening and sniffing packets for processing. Check out the next article [here](http://thepacketgeek.com/pyshark-using-the-capture-object/ "PyShark – Using the capture Object") where I go into what can be done with the capture object that is returned from these modules.