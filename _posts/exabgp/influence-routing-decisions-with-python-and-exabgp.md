---
id: 522
title: 'ExaBGP and python: Getting Started'
date: 2015-05-21T06:10:39-07:00
author: Mat
layout: post
guid: http://thepacketgeek.com/?p=522
permalink: /influence-routing-decisions-with-python-and-exabgp/
categories:
  - Coding
  - Networking
tags:
  - CSR-1000V
  - ExaBGP
  - Python
series:
  - Influence Routing Decisions with python and ExaBGP
---
<div class="seriesmeta">
  This entry is part 1 of 6 in the series <a href="https://thepacketgeek.com/series/influence-routing-decisions-with-python-and-exabgp/" class="series-26" title="Influence Routing Decisions with python and ExaBGP">Influence Routing Decisions with python and ExaBGP</a>
</div>

I&#8217;m really excited about these next few posts. I&#8217;ve been doing some research on BGP and automating routing decisions with python, which led to my discovery of <a href="https://github.com/Exa-Networks/exabgp" target="_blank" rel="noopener">ExaBGP</a>. ExaBGP is dubbed &#8220;The BGP swiss army knife&#8221;, and I&#8217;m early in my experimentation with this tool, but it seems to be a very easy way to peer with your BGP routers and control the advertisement of networks.

This post will cover basic setup of ExaBGP and peering with a router, as well as how you can tie in python to present control for the advertisement of routes. The next post will use the Flask web framework to offer a simple HTTP API for adding/removing routes. I hope the following&nbsp;posts will be along the lines of receiving the BGP UPDATE messages from peered routers to monitor and analyze advertised networks.

<!--more-->

## Step 1: Install ExaBGP

The installation is very simple, you just install via pip in your global python packages, or preferably in a virtualenv (or similar) to not interfere with your other projects:

\`$ pip3 install exabgp\` (or \`pip\` if you&#8217;re using python 2)

This should work on OS X and *nix, but I did have some problems trying to run ExaBGP&nbsp;on Windows.

## Step 2: Configure your Router

I&#8217;m using a Cisco CSR-1000V virtual router instance on the same machine I&#8217;m running python/ExaBGP on, but you can also use any router that has IP connectivity to the machine you are running ExaBGP with. Note that my CSR-1000V router has the IP address of 172.16.2.128/24, and my ExaBGP machine has an IP address of 172.16.2.1/24. Here&#8217;s my router&#8217;s very basic BGP config and local IP addresses.

<pre class="theme:dark-terminal show-lang:2 lang:default highlight:0 decode:true">CRS1#sh run | s i bgp
router bgp 65000
  bgp log-neighbor-changes
  network 10.10.0.0 mask 255.255.255.0
  neighbor 172.16.2.1 remote-as 65000
  neighbor 172.16.2.1 update-source GigabitEthernet1
CRS1#sh ip int bri
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       172.16.2.128   YES NVRAM  up                    up
Loopback0              1.1.1.1         YES NVRAM  up                    up
Loopback10             10.10.0.1       YES NVRAM  up                    up</pre>

If you run a \`show bgp summary\` command, you will see the neighbor in BGP Idle state since our ExaBGP side hasn&#8217;t tried peering yet. &nbsp;I also recommend using \`debug bgp all\` (only in a lab of course) to monitor the connection status and updates from ExaBGP:

<pre class="theme:dark-terminal lang:default highlight:0 decode:true">CRS1#show bgp summary | i 172.16.2.1
172.16.2.1      4        65000       0       0        1    0    0 00:01:53 Idle
CRS1#debug bgp all
BGP debugging is on for all address families</pre>

## Step 3: Configure ExaBGP

Now we can configure ExaBGP to peer with our router using a \`conf.ini\` file. Explanations for each directive are commented inline:

<pre class="lang:ini decode:true" title="conf.ini">neighbor 172.16.2.128 {                 # Remote neighbor to peer with
    router-id 172.16.2.1;              # Our local router-id
    local-address 172.16.2.1;          # Our local update-source
    local-as 65000;                    # Our local AS
    peer-as 65000;                     # Peer's AS
}</pre>

If we start ExaBGP now, we&#8217;ll see the neighbors peer successfully and the ExaBGP process will receive BGP UPDATE messages for any prefixes the CSR router is currently advertising. Here&#8217;s what we see when starting ExaBGP:

<pre class="theme:dark-terminal lang:default decode:true">(exabgp)packetgeek:exabgp mat$ exabgp ./conf.ini
....(configuration load info truncated)
17:01:37 | 27403  | welcome       | Thank you for using ExaBGP
17:01:37 | 27403  | version       | 4.0.2
17:01:37 | 27403  | configuration | performing reload of exabgp 4.0.2-1c737d99
17:01:37 | 27403  | reactor       | loaded new configuration successfully
17:01:37 | 27403  | reactor       | connected to peer-1 with outgoing-1 172.16.2.1-172.16.2.128
</pre>

And running \`show bgp summary\` on the router also confirms that the peering is in an Established state:

<pre class="theme:dark-terminal lang:default highlight:0 decode:true">CRS1#show bgp summary | i Neighbor|172.16.2.1
Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
172.16.2.1      4        65000       6       8       23    0    0 00:03:31        0
</pre>

Well, that&#8217;s certainly a start!

## Step 4: Write python Script to add/remove Routes

Using an <a href="https://github.com/Exa-Networks/exabgp/wiki/Controlling-ExaBGP-:-_-README-first" target="_blank" rel="noopener">example from ExaBGP</a>, we&#8217;re going to create&nbsp;a very simple python script named \`example.py\` to advertise two prefixes to our CSR router:

<pre class="lang:default decode:true" title="example.py">#!/usr/bin/env python3

from __future__ import print_function

from sys import stdout
from time import sleep

messages = [
    'announce route 100.10.0.0/24 next-hop self',
    'announce route 200.20.0.0/24 next-hop self',
]

sleep(5)

#Iterate through messages
for message in messages:
    stdout.write(message + '\n')
    stdout.flush()
    sleep(1)

#Loop endlessly to allow ExaBGP to continue running
while True:
    sleep(1)</pre>

The reason this script works is that ExaBGP monitors STDOUT while running and parses for commands such as &#8220;announce route &#8230;&#8221; and &#8220;withdraw route &#8230;&#8221;. Check out this&nbsp;<a href="https://github.com/Exa-Networks/exabgp/wiki/Controlling-ExaBGP-:-interacting-from-the-API" target="_blank" rel="noopener">full list of API commands</a>, I&#8217;ll be covering several of these in the upcoming articles. Now we just need to update our \`conf.ini\` file to run this python script upon startup of ExaBGP:

<pre class="lang:ini decode:true " title="conf.ini">process announce-routes {
    run /path/to/python3 /path/to/example.py;
    encoder json;
}

neighbor 172.16.2.128 {                 # Remote neighbor to peer with
    router-id 172.16.2.1;              # Our local router-id
    local-address 172.16.2.1;          # Our local update-source
    local-as 65000;                    # Our local AS
    peer-as 65000;                     # Peer's AS

    api {
        processes [announce-routes];
    }
}</pre>

Here&#8217;s the output from exabgp:

<pre class="theme:dark-terminal lang:default decode:true">(exabgp)packetgeek:exabgp mat$ exabgp ./conf.ini
....(configuration load info truncated)
17:11:52 | 27396  | welcome       | Thank you for using ExaBGP
17:11:52 | 27396  | version       | 4.0.2
17:11:52 | 27396  | configuration | performing reload of exabgp 4.0.2-1c737d99
17:11:52 | 27396  | reactor       | loaded new configuration successfully
17:11:52 | 27396  | reactor       | connected to peer-1 with outgoing-1 172.16.2.1-172.16.2.128
17:11:57 | 27396  | api           | route added to neighbor 172.16.2.128 local-ip 172.16.2.1 local-as 65000 peer-as 65000 router-id 172.16.2.1 family-allowed in-open : 100.10.0.0/24 next-hop self
17:11:58 | 27396  | api           | route added to neighbor 172.16.2.128 local-ip 172.16.2.1 local-as 65000 peer-as 65000 router-id 172.16.2.1 family-allowed in-open : 200.20.0.0/24 next-hop self</pre>

And after this finishes, we can confirm the new prefixes on our router by running \`show bgp\`:

<pre class="theme:dark-terminal lang:default highlight:0 decode:true ">CRS1#sh bgp | b Network
     Network          Next Hop            Metric LocPrf Weight Path
 *&gt;  10.10.0.0/24     0.0.0.0                  0         32768 i
 *&gt;i 100.10.0.0/24    172.16.2.1                    100      0 i
 *&gt;i 200.20.0.0       172.16.2.1                    100      0 i</pre>

Congrats! We just used python to inject routes into the BGP and RIB tables on a router. I know this example is a little boring since you have to preconfigure the prefixes and use sleep timers, but I&#8217;ll cover how to make this a little more interactive in the next post.