---
id: 92
title: 'Scapy p.01 &#8211; Scapy Introduction and Overview'
date: 2013-10-29T16:59:59-07:00
author: Mat
layout: post
guid: http://thepacketgeek.com/?p=92
permalink: /scapy-p-01-scapy-introduction-and-overview/
categories:
  - Coding
  - Networking
tags:
  - Scapy
series:
  - Building Network Tools with Scapy
---
<div class="seriesmeta">
  This entry is part 1 of 11 in the series <a href="https://thepacketgeek.com/series/building-network-tools-with-scapy/" class="series-13" title="Building Network Tools with Scapy">Building Network Tools with Scapy</a>
</div>

### <span style="font-size: 1em;">What is Scapy?</span> {#01}

No one can introduce Scapy better than the creator or the project himself:

> &#8220;Scapy is a powerful interactive packet manipulation program. It is able to forge or decode packets of a wide number of protocols, send them on the wire, capture them, match requests and replies, and much more. It can easily handle most classical tasks like scanning, tracerouting, probing, unit tests, attacks or network discovery&#8230;
> 
> It also performs very well at a lot of other specific tasks that most other tools can&#8217;t handle, like sending invalid frames, injecting your own 802.11 frames, combining technics (VLAN hopping+ARP cache poisoning, VOIP decoding on WEP encrypted channel, &#8230;), etc.

&#8211; Phil @  <a href="http://www.secdev.org/projects/scapy/" target="_blank">secdev.org</a>

#### <!--more-->Working with Scapy

Python is the foundation of Scapy and is the biggest dependency, but the good news is that Python can run on almost any system! If you&#8217;re using a Mac or some flavor of *nix, it&#8217;s probably already installed on your computer. Since Scapy is built on Python, you have the full functionality of Python including control blocks, conditional statements, and all of the available modules. The possiblilies are virtually limitless when it comes to building your own network tools.

#### I&#8217;m not a programmer, how easy is Scapy to use?

Scapy is used via a command-line interactive mode or inside Python scripts, but after reading through this guide you should be able to do most of the basics with Scapy. Scapy has its own syntax, so you don&#8217;t need to know much Python to get started. Python is a very friendly language to learn though and a little extra studying will widen what you can do with Scapy and your network tools.

If you&#8217;re interested in learning more about Python I highly recommend these resources:

  * <a href="http://www.codeacademy.com" target="_blank">Codeacademy</a>
  * <a href="http://www.amazon.com/Learning-Python-Mark-Lutz/dp/1449355730" target="_blank">Learning Python</a>
  * <a href="http://learnpythonthehardway.org/" target="_blank">Learn Python the Hard Way</a>