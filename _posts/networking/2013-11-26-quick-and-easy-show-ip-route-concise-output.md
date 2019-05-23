---
id: 205
title: 'Quick and Easy &#8220;show ip route&#8221; with Concise Output'
date: 2013-11-26T16:38:50-07:00
author: Mat
layout: post
guid: http://thepacketgeek.com/?p=205
permalink: /quick-and-easy-show-ip-route-concise-output/
categories:
  - Networking
tags:
  - Cisco IOS
---
For those of you Cisco IOS ninjas that can differentiate the `show ip route` table codes in your sleep, the first several lines of this command output are just a nuisance. Here&#8217;s a quick way to remove that unnecessary text from the output so you can get straight to finding out where your traffic is headed. And all with just a few additional keystrokes to the end of the command:

`# show ip route | e -`

<!--more-->Cleans up this table&#8230;

    R6#sh ip route
    Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
           D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
           N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
           E1 - OSPF external type 1, E2 - OSPF external type 2
           i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
           ia - IS-IS inter area, * - candidate default, U - per-user static route
           o - ODR, P - periodic downloaded static route, + - replicated route
    
    Gateway of last resort is not set
    
          3.0.0.0/32 is subnetted, 1 subnets
    D        3.3.3.3 [90/156160] via 10.1.36.3, 00:07:32, FastEthernet0/1
          4.0.0.0/32 is subnetted, 1 subnets
    D        4.4.4.4 [90/158720] via 10.1.36.3, 00:07:22, FastEthernet0/1
          5.0.0.0/32 is subnetted, 1 subnets
    D        5.5.5.5 [90/161280] via 10.1.36.3, 00:07:22, FastEthernet0/1
          6.0.0.0/32 is subnetted, 1 subnets
    C        6.6.6.6 is directly connected, Loopback0
          10.0.0.0/8 is variably subnetted, 4 subnets, 2 masks
    D        10.1.34.0/24 [90/30720] via 10.1.36.3, 00:07:32, FastEthernet0/1
    C        10.1.36.0/24 is directly connected, FastEthernet0/1
    L        10.1.36.6/32 is directly connected, FastEthernet0/1
    D        10.1.45.0/24 [90/33280] via 10.1.36.3, 00:07:22, FastEthernet0/1

&#8230;so it looks like this:

    R6#sh ip route | e -
    
    Gateway of last resort is not set
    
          3.0.0.0/32 is subnetted, 1 subnets
    D        3.3.3.3 [90/156160] via 10.1.36.3, 00:07:32, FastEthernet0/1
          4.0.0.0/32 is subnetted, 1 subnets
    D        4.4.4.4 [90/158720] via 10.1.36.3, 00:07:22, FastEthernet0/1
          5.0.0.0/32 is subnetted, 1 subnets
    D        5.5.5.5 [90/161280] via 10.1.36.3, 00:07:22, FastEthernet0/1
          6.0.0.0/32 is subnetted, 1 subnets
    C        6.6.6.6 is directly connected, Loopback0
          10.0.0.0/8 is variably subnetted, 4 subnets, 2 masks
    D        10.1.34.0/24 [90/30720] via 10.1.36.3, 00:07:32, FastEthernet0/1
    C        10.1.36.0/24 is directly connected, FastEthernet0/1
    L        10.1.36.6/32 is directly connected, FastEthernet0/1
    D        10.1.45.0/24 [90/33280] via 10.1.36.3, 00:07:22, FastEthernet0/1

Much cleaner, right?!?

Also, this works when you&#8217;re printing out RIBs for multiple VRFs. In just a small MPLS VPN lab the routes can quickly get out of hand, but see how much nicer this is to look through:

    R6#show ip route vrf * | e -
    
    Gateway of last resort is not set
    
          3.0.0.0/32 is subnetted, 1 subnets
    D        3.3.3.3 [90/156160] via 10.1.36.3, 00:12:56, FastEthernet0/1
          4.0.0.0/32 is subnetted, 1 subnets
    D        4.4.4.4 [90/158720] via 10.1.36.3, 00:12:46, FastEthernet0/1
          5.0.0.0/32 is subnetted, 1 subnets
    D        5.5.5.5 [90/161280] via 10.1.36.3, 00:12:46, FastEthernet0/1
          6.0.0.0/32 is subnetted, 1 subnets
    C        6.6.6.6 is directly connected, Loopback0
          10.0.0.0/8 is variably subnetted, 4 subnets, 2 masks
    D        10.1.34.0/24 [90/30720] via 10.1.36.3, 00:12:56, FastEthernet0/1
    C        10.1.36.0/24 is directly connected, FastEthernet0/1
    L        10.1.36.6/32 is directly connected, FastEthernet0/1
    D        10.1.45.0/24 [90/33280] via 10.1.36.3, 00:12:46, FastEthernet0/1
    
    Routing Table: A
    
    Gateway of last resort is not set
    
          10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
    B        10.10.10.10/32 
               [20/2] via 10.99.10.10 (SHARED), 00:11:51, FastEthernet1/0
    B        10.99.10.0/24 is directly connected, 00:11:52, FastEthernet1/0
    B        10.99.20.0/24 
               [20/2] via 10.99.10.10 (SHARED), 00:11:52, FastEthernet1/0
          192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks
    C        192.168.10.0/24 is directly connected, FastEthernet0/0
    L        192.168.10.6/32 is directly connected, FastEthernet0/0
    B     192.168.20.0/24 [200/0] via 5.5.5.5, 00:11:52
    D     192.168.100.0/24 [90/156160] via 192.168.10.7, 00:13:00, FastEthernet0/0
    D     192.168.150.0/24 [90/156160] via 192.168.10.7, 00:13:00, FastEthernet0/0
    B     192.168.200.0/24 [200/156160] via 5.5.5.5, 00:11:52
    B     192.168.250.0/24 [200/156160] via 5.5.5.5, 00:11:52
    
    Routing Table: B
    
    Gateway of last resort is not set
    
          11.0.0.0/32 is subnetted, 1 subnets
    O        11.11.11.11 [110/2] via 172.16.10.11, 00:12:08, FastEthernet1/1
          12.0.0.0/32 is subnetted, 1 subnets
    B        12.12.12.12 [200/2] via 5.5.5.5, 00:11:52
          172.16.0.0/16 is variably subnetted, 7 subnets, 2 masks
    C        172.16.10.0/24 is directly connected, FastEthernet1/1
    L        172.16.10.6/32 is directly connected, FastEthernet1/1
    B        172.16.20.0/24 [200/0] via 5.5.5.5, 00:00:02
    O        172.16.100.11/32 [110/2] via 172.16.10.11, 00:12:10, FastEthernet1/1
    O        172.16.150.11/32 [110/2] via 172.16.10.11, 00:12:10, FastEthernet1/1
    B        172.16.200.12/32 [200/2] via 5.5.5.5, 00:00:02
    B        172.16.250.12/32 [200/2] via 5.5.5.5, 00:00:02
    
    Routing Table: SHARED
    
    Gateway of last resort is not set
    
          10.0.0.0/8 is variably subnetted, 4 subnets, 2 masks
    O        10.10.10.10/32 [110/2] via 10.99.10.10, 00:12:17, FastEthernet1/0
    C        10.99.10.0/24 is directly connected, FastEthernet1/0
    L        10.99.10.6/32 is directly connected, FastEthernet1/0
    O        10.99.20.0/24 [110/2] via 10.99.10.10, 00:12:17, FastEthernet1/0
    B     192.168.10.0/24 is directly connected, 00:11:54, FastEthernet0/0
    B     192.168.20.0/24 [200/0] via 5.5.5.5, 00:11:54
    B     192.168.100.0/24 
               [20/156160] via 192.168.10.7 (A), 00:11:54, FastEthernet0/0
    B     192.168.150.0/24 
               [20/156160] via 192.168.10.7 (A), 00:11:54, FastEthernet0/0
    B     192.168.200.0/24 [200/156160] via 5.5.5.5, 00:11:54
    B     192.168.250.0/24 [200/156160] via 5.5.5.5, 00:11:56