---
id: 459
title: 'OnePK &#8211; Configuring and Assigning an ACL'
date: 2014-11-22T08:51:54-07:00
author: Mat
layout: post
guid: http://thepacketgeek.com/?p=459
permalink: /onepk-configuring-and-assigning-an-acl/
categories:
  - Coding
  - Networking
tags:
  - OnePK
  - Python
series:
  - Getting started with Cisco OnePK
---
<div class="seriesmeta">
  This entry is part 5 of 5 in the series <a href="https://thepacketgeek.com/series/cisco-onepk/" class="series-24" title="Getting started with Cisco OnePK">Getting started with Cisco OnePK</a>
</div>

During my research of the ACL module of OnePK I found some interesting behaviors. I think this feature isn&#8217;t fully cooked as it has some definite room for improvement. The ACL module (found in \`onep.policy\`) doesn&#8217;t have a way to interact with existing ACLs and ACEs on an IOS device. The ACLs seem to be intended for on-the-fly ACL configuration but with a couple caveats:

  * Applying an ACL to an interface overwrites the existing ACL for that direction. Interfaces must play by the rule: &#8216;One ACL per direction&#8217;.
  * Make sure to use \`OnepLifetime.ONEP_PERSISTENT\` to keep ACLs applied after the OneP session is disconnected.

First I&#8217;ll cover ACL management with OneP, and then I&#8217;ll use the \`onep.vty\` module to show how to workaround some of the shortcomings.

<!--more-->

### Creating and Applying an ACL On-the-Fly

A use case that fits in well with OnePK ACL management is creating dynamic ACLs based on input from a separate application such as a IDS/IPS system. Let&#8217;s use this scenario for the first example: Our OnePK app received an alert (possibly a syslog or SNMP trap) that a device on the network is receiving an attack using a SSH exploit and we need to stop the bogus traffic.  We know that SSH uses TCP port 22 and also that the end host receiving the attack has an IP address of 10.10.20.15/24.  Here&#8217;s one way we can create an ACL to block that traffic:

<pre class="lang:default decode:true">from onep_connect import connect
from onep.core.util import OnepConstants
from onep.policy import Acl, L3Acl, L3Ace

# Connect to a router inline with the attacker's traffic
ne = connect('10.211.55.200', 'admin', 'admin')

#specify the interface towards the attacker
interface = ne.get_interface_by_name('thub2')

#  Create a IPv4 L3 ACL
l3_acl = L3Acl(ne, OnepConstants.OnepAddressFamilyType.ONEP_AF_INET, L3Acl.OnepLifetime.ONEP_PERSISTENT)

# Create ACE that matches the attacker's traffic
l3_ace_10 = L3Ace(10, False)  #False == deny
l3_ace_10.protocol = OnepConstants.AclProtocol.TCP
l3_ace_10.set_src_prefix_any()
l3_ace_10.dst_prefix = '10.10.20.15'
l3_ace_10.dst_prefix_len = 32            
l3_ace_10.set_dst_port_range(22,22)    #SSH Port

# Create ACE to allow all other traffic
l3_ace_20 = L3Ace(20, True)  #True == permit
l3_ace_20.protocol = OnepConstants.AclProtocol.ALL
l3_ace_20.set_src_prefix_any()
l3_ace_20.set_dst_prefix_any()

# Add both ACEs to the ACL
l3_acl.add_ace(l3_ace_10)
l3_acl.add_ace(l3_ace_20)

# Apply the ACL to the interface
l3_acl.apply_to_interface(interface, Acl.Direction.ONEP_DIRECTION_IN)

# End the onep session (ACL remains because we used a persistent lifetime)
ne.disconnect()</pre>

There isn&#8217;t a lot of logging provided on the IOS device to tell us that the ACL was applied, but we can run a \`show ip interface gi2\` command at the enable prompt to see the following confirmation:

<pre class="toolbar:2 striped:false lang:sh highlight:0 decode:true ">CSR1#sh ip interface gi2 | i access list
  Outgoing Common access list is not set
  Outgoing access list is not set
  Inbound Common access list is not set
  Inbound  access list is onep-acl-18</pre>

Boom, there&#8217;s our dynamic ACL on the inbound direction of the &#8216;gi2&#8217; interface. We could apply this interface in both directions if we used \`Acl.Direction.ONEP\_DIRECTION\_BOTH\` in the \`.apply\_to\_interface()\` method. Also, you guessed it, we can use \`Acl.Direction.ONEP\_DIRECTION\_OUT\` for traffic leaving that interface also.  This ACL could also be applied to multiple interfaces (more than one Internet facing interface?) like this:

<pre class="lang:default decode:true "># Apply the ACL to multiple interfaces
l3_acl.apply_to_interface(ne.get_interface_by_name('gi2'), Acl.Direction.ONEP_DIRECTION_IN)
l3_acl.apply_to_interface(ne.get_interface_by_name('fa0/1'), Acl.Direction.ONEP_DIRECTION_IN)</pre>

Depending on our expectations for the life of the ACL, we can have it automatically removed at the end of the onep session by specifying  \`L3Acl.OnepLifetime.ONEP_TRANSIENT\` during the creation of the L3Acl Object.

To see a more in-depth example that includes syslog parsing and a bi-directional ACE, check it out on the <a title="GitHub - ACL from Syslog" href="https://github.com/thepacketgeek/cisco-onepk-python-examples/blob/master/scripts/create_ACL_from_syslog.py" target="_blank">GitHub repo here</a>.

### Viewing and Modifying Existing ACLs

Support for interacting with existing ACLs isn&#8217;t directly built-in to OnePK, but we can use the \`onep.vty\` module to achieve some pretty close capabilities. Here&#8217;s a simple way to view existing ACLs:

<pre class="lang:default decode:true">from one_connect import connect
from onep.vty.util import VtyHelper

ne = connect('1.1.1.1', 'admin', 'admin')
ne_vty = VtyHelper(ne)

ne_vty.show('ip access-list')
</pre>

Which gives the output:

<pre class="toolbar:2 striped:false lang:sh highlight:0 decode:true ">Extended IP access list DEMO
    10 permit ip 10.10.20.0 0.0.0.255 any
    20 deny tcp any any eq smtp
    30 permit tcp 192.168.10.0 0.0.0.255 any range www 443
Extended IP access list onep-acl-18
    10 permit tcp any any range 22 22</pre>

Very cool, there&#8217;s our dynamically created ACL from the previous example along with an existing ACL on the router. This is returned to us as a string but some fancy parsing could be done in order to confirm compliance to policies. Using that same \`VtyHelper\` object, we can also modify the DEMO ACL by adding an ACE using the \`.config()\` method:

<pre class="lang:default decode:true">from one_connect import connect
from onep.vty.util import VtyHelper

ne = connect('1.1.1.1', 'admin', 'admin')
ne_vty = VtyHelper(ne)

ne_vty.vty_exec('conf t')
ne_vty.vty_exec('ip acce ext DEMO')
ne_vty.vty_exec('15 deny udp any any')
ne_vty.vty_exec('no 20')
ne_vty.vty_exec('end')

ne_vty.show('ip access-list')</pre>

And we can see our added sequence #15 ACE and removed #20 sequence ACE in the DEMO ACL:

<pre class="toolbar:2 striped:false lang:sh mark:3 highlight:0 decode:true">Extended IP access list DEMO
    10 permit ip 10.10.20.0 0.0.0.255 any
    15 deny udp any any
    20 deny tcp any any eq smtp
    30 permit tcp 192.168.10.0 0.0.0.255 any range www 443
Extended IP access list onep-acl-18
    10 permit tcp any any range 22 22</pre>

With both of these ACL creation technique, don&#8217;t forget to save the config on the device with the VtyHelper \`.commit()\` method.

&#8212;

As you can see, dynamic ACL creation is very powerful especially since you can manipulate existing ACLs with the \`VtyHelper\` object. I think this is a good supplement in places where the onep SDK doesn&#8217;t have full feature coverage. It certainly does open the door to any configuration that can be done via CLI.