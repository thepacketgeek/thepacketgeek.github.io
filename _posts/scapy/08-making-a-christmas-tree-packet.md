---
title: Scapy p.08 – Making a Christmas Tree Packet
date: 2018-10-29T16:59:52-07:00
author: Mat
layout: post
categories:
  - scapy
---
We&#8217;ve doing a lot of packet sniffing, analysis, and even some basic packet crafting of our own. With the ICMP packets we created, we only set the destination we wanted to use and let Scapy take care of the rest.

#### Taking Control of Protocol Fields

I want to show you how to take a bit more control over the packet creation process by creating a [TCP Christmas Tree packet](http://en.wikipedia.org/wiki/Christmas_tree_packet). I&#8217;ll let you read the details, just know that the name of this packet comes from every TCP header flag bit turned on (set to 1), so it can be said the packet is &#8220;lit up like a Christmas Tree.&#8221; <!--more-->Here&#8217;s how we can build this with Scapy:

<pre class="lang:default decode:true ">#! /usr/bin/env python3

from random import randint
from scapy.all import IP, TCP, send

# Create the skeleton of our packet
template = IP(dst="172.16.20.10")/TCP()

# Start lighting up those bits!
template[TCP].flags = 'UFP'

# Create a list with a large number of packets to send
# Each packet will have a random TCP dest port for attack obfuscation
xmas = []
for pktNum in range(0,100):
  xmas.extend(template)
  xmas[pktNum][TCP].dport = randint(1,65535)

# Send the list of packets
send(xmas)


============================Console Output:===========================
....................................................................................................
Sent 100 packets.</pre>

Although we don&#8217;t get much output from the `send()` function, and no option for the `prn` argument, we can sniff and see what happened:

<div id="attachment_113" style="width: 660px" class="wp-caption aligncenter">
  <a href="http://thepacketgeek.com/wp-content/uploads/2013/10/08-xmas-tree-packets.png"><img aria-describedby="caption-attachment-113" class="size-large wp-image-113 " src="//thepacketgeek.com/wp-content/uploads/2013/10/08-xmas-tree-packets-1024x677.png" alt="08-xmas-tree-packets" width="650" height="429" srcset="https://thepacketgeek.com/wp-content/uploads/2013/10/08-xmas-tree-packets-1024x677.png 1024w, https://thepacketgeek.com/wp-content/uploads/2013/10/08-xmas-tree-packets-300x198.png 300w, https://thepacketgeek.com/wp-content/uploads/2013/10/08-xmas-tree-packets.png 1268w" sizes="(max-width: 650px) 100vw, 650px" /></a>
  
  <p id="caption-attachment-113" class="wp-caption-text">
    Wireshark sniff showing several xmas tree packets and the TCP header with our bits set
  </p>
</div>

Woohoo! Look how awesome we are! Make sure to look through that script so you can see what we&#8217;re doing. We want to send random TCP ports in our packet, so we have to make an array of packets, each with a different TCP destination port. You could also randomize the source port or any other field using the technique I did in that script.