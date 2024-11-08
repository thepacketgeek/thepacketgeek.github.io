+++
title = "Scapy p.01"
description = "Scapy Introduction and Overview"
#date = 2013-10-29
date = 2019-05-11
author = "Mat"
weight = 100
in_search_index = true
aliases = ["/scapy-p-01-scapy-introduction-and-overview/"]

[taxonomies]
tags = ["scapy", "python"]
+++

### What is Scapy?

No one can introduce Scapy better than the creator of the project himself:

> Scapy is a powerful interactive packet manipulation program. It is able to forge or decode packets of a wide number of protocols, send them on the wire, capture them, match requests and replies, and much more. It can easily handle most classical tasks like scanning, tracerouting, probing, unit tests, attacks or network discovery
> 
> It also performs very well at a lot of other specific tasks that most other tools can't handle, like sending invalid frames, injecting your own 802.11 frames, combining technics (VLAN hopping+ARP cache poisoning, VOIP decoding on WEP encrypted channel, ...), etc.

- Phil @  <a href="http://www.secdev.org/projects/scapy/" target="_blank">secdev.org</a>

<!-- more -->
## Working with Scapy

Python is the foundation of Scapy and is the biggest dependency, but the good news is that Python can run on almost any system! If you're using a Mac or some flavor of *nix, it's probably already installed on your computer. Since Scapy is built on Python, you have the full functionality of Python including control blocks, conditional statements, and all of the available modules. The possiblilies are virtually limitless when it comes to building your own network tools.

## I'm not a programmer, how easy is Scapy to use?

Scapy is used via a command-line interactive mode or inside Python scripts, but after reading through this guide you should be able to do most of the basics with Scapy. Scapy has its own syntax, so you don't need to know much Python to get started. Python is a very friendly language to learn though and a little extra studying will widen what you can do with Scapy and your network tools.

If you're interested in learning more about Python I highly recommend these resources:

  * <a href="http://www.codeacademy.com" target="_blank">Codeacademy</a>
  * <a href="http://www.amazon.com/Learning-Python-Mark-Lutz/dp/1449355730" target="_blank">Learning Python</a>
  * <a href="http://learnpythonthehardway.org/" target="_blank">Learn Python the Hard Way</a>