---
id: 328
title: Writing Packets to Trace File with Scapy
date: 2015-05-20T19:42:33-07:00
author: Mat
layout: post
guid: http://thepacketgeek.com/?p=328
permalink: /writing-packets-to-trace-file-with-scapy/
categories:
  - Coding
tags:
  - Scapy
---
This is just a quick follow-up post to accompany the previous <a href="//thepacketgeek.com/importing-packets-from-trace-files/" target="_blank" rel="noopener">Importing packets from trace files with scapy</a> post. So you&#8217;ve sniffed or generated some packets with scapy and it&#8217;s time to write them to file to analyze and double-check your work. Here&#8217;s a simple example of how to save those packets.

<pre class="lang:default mark:6 decode:true  ">localhost:~ packetgeek$ scapy
&gt;&gt;&gt; packets = sniff(count=10)
&gt;&gt;&gt; packets
&lt;Sniffed: TCP:0 UDP:3 ICMP:0 Other:7&gt;
&gt;&gt;&gt; wrpcap('sniffed.pcap', packets)</pre>

Tada!  That&#8217;s it. There&#8217;s no options or special functions, you probably should do your packet processing before you write the packets to file.