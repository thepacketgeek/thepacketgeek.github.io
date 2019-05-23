---
id: 654
title: Rethinking Security Policies for IPv6
date: 2015-07-14T16:58:03-07:00
author: Mat
layout: post
guid: https://thepacketgeek.com/?p=654
permalink: /rethinking-security-policies-for-ipv6/
categories:
  - Networking
---
You&#8217;ve been assigned the task of deploying adding IPv6 to your network so you can finally feel the dual stack warm and fuzzies. The process for doing so should be something along the lines of:

  * Put some of those just-too-long-to-memorize IPv6 addresses on your router interfaces.
  * Choose an IPv6 capable routing protocol to share the prefixes throughout your network.
  * Clone your IPv4 security policies and ACLs to permit hosts on the public Internet to access your servers.

Easy, right? Those first two items are a step in the right direction, but the third item is most likely going to cause you some trouble. I&#8217;m guessing that somewhere in an ACL on your network there&#8217;s a couple of statements that look similar to this:

<pre class="theme:dark-terminal lang:default decode:true">ip access-list extended INTERNET_IN
    permit tcp any 172.16.10.10 0.0.0.1 eq www
    permit tcp any host 172.16.10.12 eq ftp</pre>

The ACL is probably not that simple, but we&#8217;ll keep it short for discussion&#8217;s sake. The most important part here is that TCP port 80 traffic is allowed to two **hosts** by IP address destination, and TCP port 21 is allowed to one **host** also by IP address destination. These statements work because you know exactly which IP addresses are configured on the servers and they&#8217;re most likely the only IP address configured on the servers&#8217; NICs.  This is where IPv6 changes everything you know about security policies.

<!--more-->

### IPv6 Addressing Review

Chances are you&#8217;re familiar with IPv6 and the 128-bit addresses, 64-bit subnet masks allowing for an effectively infinite amount of hosts per subnet, and MAC address based IPv6 address assignment. Here are a few more fun facts about IPv6:

  * IPv6 enabled interfaces are not only allowed to have multiple IPv6 addresses, it&#8217;s encouraged.
  * Link-local addresses are used for discover and communicate local routers and neighbors.
  * ICMPv6 RA learned prefix addresses allow a host to self-assign addresses for Global IPv6 communication. 
      * Address assignment is typically based on the MAC address of the NIC.
      * In order to help with privacy concerns, hosts even self-assign <a href="https://tools.ietf.org/html/rfc3041" target="_blank">random temporary addresses</a>.

It&#8217;s completely possible to have an interface with all these addresses:

<pre class="theme:dark-terminal lang:default decode:true">thepacketgeek@computer:~$ ifconfig
eth0      Link encap:Ethernet  HWaddr 00:11:22:aa:bb:cc 
          inet addr:172.10.10.1  Bcast:172.10.10.255  Mask:255.255.255.0

          inet6 addr: 2001:8ea4:ca77::25f3:1922:abcd/64 Scope:Global
          inet6 addr: 2001:8ea4:ca77::0311:22ff:feaa:bbcc/64 Scope:Global
          inet6 addr: fe80::0311:22ff:feaa:bbcc/64 Scope:Link
(truncated..)
</pre>

When it&#8217;s time to convert that web traffic ACL above to IPv6, which IPv6 address in the \`ifconfig\` output will you put as your destination address? Also, assuming you pick one that works now, what happens later when a self-assigned address changes? I have an idea on how to combat this issue and keep your security policy management feasible.

### Group Similar Services into the Same Prefix

Instead of allowing traffic to specific hosts by destination IPv6 address I propose grouping your services into IPv6 subnets so that each service will have its own prefix. Using this method your IPv6 ACL might look something like:

<pre class="theme:dark-terminal lang:default decode:true">ipv6 access-list INTERNET_IN_v6
  permit tcp any 2001:a::/64 eq www
  permit tcp any 2001:b::/64 eq ftp</pre>

<img class="aligncenter wp-image-687" src="//thepacketgeek.com/wp-content/uploads/2015/07/2015-07-14-09_25_48-IPv6-ACL_-Lucidchart.png" alt="Two subnets and servers" width="450" height="193" srcset="https://thepacketgeek.com/wp-content/uploads/2015/07/2015-07-14-09_25_48-IPv6-ACL_-Lucidchart.png 583w, https://thepacketgeek.com/wp-content/uploads/2015/07/2015-07-14-09_25_48-IPv6-ACL_-Lucidchart-300x129.png 300w" sizes="(max-width: 450px) 100vw, 450px" /> In this example your HTTP servers are all located within the \`2001:a::/64\` prefix and FTP servers are located in the \`2001:b::/64\` prefix. The router(s) acting as a gateway for these networks will have reachability to both subnets and you can still manage connectivity between subnets. This is much easier to manage from a security policy point of view, but it also assumes that your services are running on separate servers. If your services are running on the same box (HTTP, HTTPS, FTP, etc.), managing access of applications to specific hosts will again be much more difficult to manage and keep secure. The strategy of dedicated prefixes for each service may not work in your environment which is why I&#8217;m expanding on this idea in the next section.

### Use IPv6 Address Binding to Host Services in Specific Prefixes

Since IPv6 allows multiple IPv6 addresses per NIC, why not take advantage of that feature to help make your security policy planning easier? Many applications allow you to bind to a specific IPv4/IPv6 address; <a href="http://httpd.apache.org/docs/2.2/bind.html" target="_blank">here&#8217;s how you would do so with Apache</a> for example. The idea here is to have multiple L3 interfaces on the gateway device and a multi-homed server or multiple prefixes on one L3 interface on the gateway device:

<div id="attachment_688" style="width: 460px" class="wp-caption aligncenter">
  <img aria-describedby="caption-attachment-688" class="wp-image-688" src="//thepacketgeek.com/wp-content/uploads/2015/07/2015-07-14-09_26_01-IPv6-ACL_-Lucidchart.png" alt="Two ints and one server" width="450" height="158" srcset="https://thepacketgeek.com/wp-content/uploads/2015/07/2015-07-14-09_26_01-IPv6-ACL_-Lucidchart.png 630w, https://thepacketgeek.com/wp-content/uploads/2015/07/2015-07-14-09_26_01-IPv6-ACL_-Lucidchart-300x105.png 300w" sizes="(max-width: 450px) 100vw, 450px" />
  
  <p id="caption-attachment-688" class="wp-caption-text">
    Two interfaces, one server
  </p>
</div>

<div id="attachment_689" style="width: 460px" class="wp-caption aligncenter">
  <img aria-describedby="caption-attachment-689" class="wp-image-689" src="//thepacketgeek.com/wp-content/uploads/2015/07/2015-07-14-09_26_10-IPv6-ACL_-Lucidchart.png" alt="One Int and one server" width="450" height="153" srcset="https://thepacketgeek.com/wp-content/uploads/2015/07/2015-07-14-09_26_10-IPv6-ACL_-Lucidchart.png 632w, https://thepacketgeek.com/wp-content/uploads/2015/07/2015-07-14-09_26_10-IPv6-ACL_-Lucidchart-300x102.png 300w" sizes="(max-width: 450px) 100vw, 450px" />
  
  <p id="caption-attachment-689" class="wp-caption-text">
    One interface, one server
  </p>
</div>

The first example could also use a trunk to the server and SVIs with each IPv6 prefix, although this would mean more interfaces to manage on both the router/switch and the server.

In the second example the host will learn about both prefixes from the ICMPv6 RAs sent by the gateway. The only extra configuration required is within the application running in order to bind to a specific IPv6 address. The address binding can most likely be automated by the config provisioning of the server, here are some options to take full advantage of this solution:

  * Bind to a temporary/random IPv6 address in the correct prefix to hide the MAC address of the server from the Internet. DNS would be used to make sure external hosts will be able to find the correct endpoint IPv6 address.
  * Add and bind an IPv6 anycast address in the correct prefix to allow for load-balancing of the application across multiple servers.

* * *

I know this idea is different and maybe not what you&#8217;re used to with IPv4 where we&#8217;ve learned that only one subnet should ever exist on a L2 segment. Please let me know what you think in the comments below. I&#8217;m open to hearing why this is a horrible idea along with comments on how to improve better utilize IPv6 addressing for security purposes.