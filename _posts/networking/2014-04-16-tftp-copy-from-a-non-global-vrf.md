---
id: 219
title: TFTP copy from a non-global VRF
date: 2014-04-16T13:37:43-07:00
author: Mat
layout: post
guid: http://thepacketgeek.com/?p=219
permalink: /tftp-copy-from-a-non-global-vrf/
categories:
  - IOS
tags:
  - CSR-1000V
---
I&#8217;ve been running a Cisco CSR-1000v box on my Mac in Parallels for a bit now. I&nbsp;love the convenience of being&nbsp;able to test on a real IOS XE device anywhere I am (airplane, coffee shop, maybe even my office)! I&#8217;ve been running the CSR-1000v since version 3.09 and I wanted to upgrade to 3.12.0S since it&#8217;s got some <a title="Cisco - 3.12.0S Release Notes" href="http://www.cisco.com/c/en/us/td/docs/routers/csr1000/release/notes/csr1000v_3Srn.html#pgfId-3130478" target="_blank">new features, bug fixes</a>, and most importantly (for me) a lower memory footprint. I downloaded the <a title="Cisco - CSR-1000v 3.12.0S Downloads" href="http://software.cisco.com/download/release.html?mdfid=284364978&flowid=39582&softwareid=282046477&release=3.12.0S&relind=AVAILABLE&rellifecycle=ED&reltype=latest" target="_blank">new .bin file</a> and proceeded to try and upgrade the image as I would any physical device, with TFTP. Well, here&#8217;s how well that went:

    CSR2#copy tftp://10.211.55.2/csr1000v-universalk9.03.12.00.S.154-2.S-std.SPA.bin bootflash:
    Destination filename [csr1000v-universalk9.03.12.00.S.154-2.S-std.SPA.bin]?
    Accessing tftp://10.211.55.2/csr1000v-universalk9.03.12.00.S.154-2.S-std.SPA.bin...
    %Error opening tftp://10.211.55.2/csr1000v-universalk9.03.12.00.S.154-2.S-std.SPA.bin (Timed out)

<!--more-->

Let me introduce you to my basic setup on my laptop: 2 CSR-1000v appliances with 2 vNICs each. 1 vNIC is connected to the Parallels shared network for management, the other is connected the Parallels Host-Only network for Inter-CSR connectivity. The management interface is in a MGT VRF since this is the default configuration of the CSR-1000v.

    CSR2#sh ip int bri
    Interface            IP-Address     OK? Method Status        Protocol
    GigabitEthernet1     10.211.55.201  YES NVRAM  up            up
    GigabitEthernet2     192.168.32.2   YES NVRAM  up            up
    Loopback0            2.2.2.2        YES NVRAM  up            up
    CSR2#sh ip vrf
      Name           Default RD       Interfaces
      MGT            2.2.2.2:1        Gi1

The instant I started typing `ping` into the CLI, I knew what my problem was. I had no connectivity in the default VRF, illustrated here:

    CSR2#ping vrf MGT 10.211.55.2
    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 10.211.55.2, timeout is 2 seconds:
    .....
    Success rate is 0 percent (0/5)
    CSR2#ping vrf MGT 10.211.55.2
    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 10.211.55.2, timeout is 2 seconds:
    !!!!!
    Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms

Ok, well at least I knew what my issue was. Question is, does TFTP have a `vrf` option?

<pre class="lang:default decode:true ">CSR2#copy tftp://10.211.55.2/file.example bootflash: 
    &lt;cr&gt;</pre>

&nbsp;

Nope. After doing a little digging around though I did find this:

    CSR2(config)#ip tftp source-interface gi1 

And guess what, problem solved! TFTP will now use the specified source-interface which happens to belong to the correct MGT VRF. I didn&#8217;t have to do any VRF route leaking changing of my VRF config. Let the upgrade begin!

    CSR2#copy tftp://10.211.55.2/csr1000v-universalk9.03.12.00.S.154-2.S-std.SPA.bin bootflash:
    Destination filename [csr1000v-universalk9.03.12.00.S.154-2.S-std.SPA.bin]? 
    Accessing tftp://10.211.55.2/csr1000v-universalk9.03.12.00.S.154-2.S-std.SPA.bin...
    Loading csr1000v-universalk9.03.12.00.S.154-2.S-std.SPA.bin from 10.211.55.2 (via GigabitEthernet1): !!!!!!!!!!!!!!! [truncated]
    [OK - 292592380 bytes]
    
    292592380 bytes copied in 195.114 secs (1499597 bytes/sec)
    

&nbsp;