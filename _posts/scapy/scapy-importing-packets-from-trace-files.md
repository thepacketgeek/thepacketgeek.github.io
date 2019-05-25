---
title: Importing packets from trace files with scapy
date: 2018-09-25T22:23:59-07:00
author: Mat
layout: post
categories:
  - scapy
---
Scapy is amazingly flexible when it comes to creating packets, but in some cases you may want to mangle or change packets that you&#8217;ve sniffed and saved in a trace file. Scapy currently supports .cap and .pcap files, but unfortunately no .pcapng files (yet&#8230;).  Reading these files are possible through the `rdpcap()` function:

<pre class="lang:default decode:true">localhost:~ packetgeek$ scapy
&gt;&gt;&gt; packets = rdpcap('IBGP_adjacency.cap')
&gt;&gt;&gt; packets
&lt;IBGP_adjacency.cap: TCP:17 UDP:0 ICMP:0 Other:0&gt;</pre>

*Thanks to <a title="PacketLife.net" href="http://packetlife.net/captures/protocol/bgp/" target="_blank" rel="noopener">packetlife.net</a> for the iBGP capture found <a title="iBGP_adjacency.cap" href="https://www.cloudshark.org/captures/00249be4441f" target="_blank" rel="noopener">here.</a>

<!--more-->

Then we can view, edit, change the packets like we could with any other packets that were sniffed or created with scapy.

<pre class="lang:default decode:true">&gt;&gt;&gt; packets.summary()
Ether / IP / TCP 4.4.4.4:11965 &gt; 3.3.3.3:bgp S / Padding
Ether / IP / TCP 3.3.3.3:bgp &gt; 4.4.4.4:11965 SA / Padding
Ether / IP / TCP 4.4.4.4:11965 &gt; 3.3.3.3:bgp A / Padding
Ether / IP / TCP 4.4.4.4:11965 &gt; 3.3.3.3:bgp PA / Raw
Ether / IP / TCP 3.3.3.3:bgp &gt; 4.4.4.4:11965 PA / Raw
Ether / IP / TCP 4.4.4.4:11965 &gt; 3.3.3.3:bgp PA / Raw
Ether / IP / TCP 3.3.3.3:bgp &gt; 4.4.4.4:11965 PA / Raw
Ether / IP / TCP 4.4.4.4:11965 &gt; 3.3.3.3:bgp PA / Raw
Ether / IP / TCP 3.3.3.3:bgp &gt; 4.4.4.4:11965 PA / Raw
Ether / IP / TCP 4.4.4.4:11965 &gt; 3.3.3.3:bgp PA / Raw
Ether / IP / TCP 4.4.4.4:11965 &gt; 3.3.3.3:bgp PA / Raw
Ether / IP / TCP 3.3.3.3:bgp &gt; 4.4.4.4:11965 A / Padding
Ether / IP / TCP 3.3.3.3:bgp &gt; 4.4.4.4:11965 PA / Raw
Ether / IP / TCP 4.4.4.4:11965 &gt; 3.3.3.3:bgp A / Padding
Ether / IP / TCP 4.4.4.4:11965 &gt; 3.3.3.3:bgp PA / Raw
Ether / IP / TCP 3.3.3.3:bgp &gt; 4.4.4.4:11965 PA / Raw
Ether / IP / TCP 4.4.4.4:11965 &gt; 3.3.3.3:bgp A / Padding
&gt;&gt;&gt;
&gt;&gt;&gt; # Changing IP destination of BGP OPEN message
&gt;&gt;&gt; packets[3][IP].dst='1.1.1.1'
&gt;&gt;&gt; packets[3].summary()
'Ether / IP / TCP 4.4.4.4:11965 &gt; 1.1.1.1:bgp PA / Raw'</pre>

## Reading .pcaps with a custom function

We can also use scapy&#8217;s `sniff()` function to read packets from a .pcap file using the `offline` argument as show here:

<pre class="lang:default decode:true">&gt;&gt;&gt; packets = sniff(offline='IBGP_adjacency.cap')
&gt;&gt;&gt;
&gt;&gt;&gt; # packets is now the same list as in the previous example
&gt;&gt;&gt; packets
&lt;Sniffed: TCP:17 UDP:0 ICMP:0 Other:0&gt;</pre>

This will allow us to use the prn() function to import the packets with custom functions, as covered in <a title="Scapy Sniffing with Custom Actions, Part 1" href="scapy-sniffing-with-custom-actions-part-1/" target="_blank" rel="noopener">this post</a>. Here we can count the packets as we import them from the .pcap file:

<pre class="lang:default decode:true ">&gt;&gt;&gt; packetCount = 0
&gt;&gt;&gt; def customAction(packet):
...    return f"{packet[0][1].src} ==&gt; {packet[0][1].dst}"
...
&gt;&gt;&gt; sniff(offline='IBGP_adjacency.cap', prn=customAction)
4.4.4.4 ==&gt; 3.3.3.3
3.3.3.3 ==&gt; 4.4.4.4
4.4.4.4 ==&gt; 3.3.3.3
4.4.4.4 ==&gt; 3.3.3.3
3.3.3.3 ==&gt; 4.4.4.4
4.4.4.4 ==&gt; 3.3.3.3
3.3.3.3 ==&gt; 4.4.4.4
4.4.4.4 ==&gt; 3.3.3.3
3.3.3.3 ==&gt; 4.4.4.4
4.4.4.4 ==&gt; 3.3.3.3
4.4.4.4 ==&gt; 3.3.3.3
3.3.3.3 ==&gt; 4.4.4.4
3.3.3.3 ==&gt; 4.4.4.4
4.4.4.4 ==&gt; 3.3.3.3
4.4.4.4 ==&gt; 3.3.3.3
3.3.3.3 ==&gt; 4.4.4.4
4.4.4.4 ==&gt; 3.3.3.3
&lt;Sniffed: TCP:17 UDP:0 ICMP:0 Other:0&gt;</pre>

##  Manipulating packets during import

We can also make a more useful function during import that changes the packets as we import.  This example below just will clear the MAC addresses to keep physical device information anonymous, but with the power of python and a little imagination the possibilities are endless!

<pre class="lang:default decode:true">&gt;&gt;&gt; # create packet list
&gt;&gt;&gt; packets = []
&gt;&gt;&gt;
&gt;&gt;&gt; # custom action function
&gt;&gt;&gt; def customAction(packet):
...    packet[Ether].dst = "00:11:22:aa:bb:cc"
...    packet[Ether].src = "00:11:22:aa:bb:cc"
...
&gt;&gt;&gt; for sniffed_packet in sniff(offline='IBGP_adjacency.cap', prn=customAction):
...     packets.append(sniffed_packet)
&gt;&gt;&gt;
&gt;&gt;&gt; # See edited MAC addresses in packet 'Ether' layer
&gt;&gt;&gt; packets[0][Ether].summary()
'Ether / IP / TCP 4.4.4.4:11965 &gt; 3.3.3.3:bgp S / Padding'
&gt;&gt;&gt; packets[0][Ether].show()
&gt;&gt;&gt; packets[0][Ether].show()
###[ Ethernet ]###
  dst= 00:11:22:aa:bb:cc
  src= 00:11:22:aa:bb:cc
  type= 0x800
...
... (truncated for brevity)
</pre>

Check back soon for the next post in this series where we cover writing packets (sniffed, created, or mangled with Scapy) back to .cap/.pcap files!