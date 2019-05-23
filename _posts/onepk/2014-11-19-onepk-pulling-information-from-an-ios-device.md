---
id: 422
title: 'OnePK &#8211; Pulling information from an IOS device'
date: 2014-11-19T09:34:06-07:00
author: Mat
layout: post
guid: http://thepacketgeek.com/?p=422
permalink: /onepk-pulling-information-from-an-ios-device/
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
  This entry is part 3 of 5 in the series <a href="https://thepacketgeek.com/series/cisco-onepk/" class="series-24" title="Getting started with Cisco OnePK">Getting started with Cisco OnePK</a>
</div>

The first logical step in controlling your network with python is to have a way of pulling important information from your network devices. This post will cover the gathering of interface and routing table information. In order to keep examples short and to the point, they  will use the `one_connect.py` script as a connection module as discussed in the <a title="OnePK - Connection to a Network Element" href="http://thepacketgeek.com/onepk-connecting-to-a-network-element" target="_blank">second post</a> of the series. My goal for these examples isn&#8217;t to show what you should have in your OnePk apps, but rather to introduce you to basic concepts so you can see how the different pieces work and how you might be able to get the information necessary in your applications.<!--more-->

### Collect Interface IP and IPv6 addresses

This first example is similar to a `show ip interface brief` and `show ipv6 interface brief` at the same time. The script finds all interfaces on the device and prints out the address list. I&#8217;ll explain simple modifications you can make after you see the example below (you can also see it on <a title="GitHub - Print Interface IPs" href="https://github.com/thepacketgeek/cisco-onepk-python-examples/blob/master/scripts/print_all_interface_IPs.py" target="_blank">GitHub here</a>):

<pre class="lang:default decode:true">from onep_connect import connect
from onep.interfaces import InterfaceFilter

# Connect using IOS device connection values
ne = connect('1.1.1.1', 'username', 'password')

try:  
    #Create Interface Filter and print interface list
    if_filter = InterfaceFilter(interface_type=1)
    for interface in ne.get_interface_list(if_filter):
        print '%s: %s' % (interface.name, interface.get_address_list())
	
finally:
    # Finally have the application disconnect from the Network Element  
    ne.disconnect() 

</pre>

The output of this will look similar (but with different IP addresses) to this:

<pre class="toolbar:2 striped:false lang:sh highlight:0 decode:true ">Loopback0: ['1.1.1.1', '2001:100::1', 'FE80::AA:BB:CC:DD']
GigabitEthernet1: ['10.211.55.200']
GigabitEthernet2: ['192.168.56.1', '2001:56::1', 'FE80::111:222:333:444']
GigabitEthernet3: []</pre>

For this example, we needed to create an `InterfaceFilter()` object which takes in either an interface name, or an interface type to search for. An `interface_type` of 1 represents all interfaces types, so I&#8217;m really just telling the `get_interface)list()` method to return a list of all interface objects on the Network Element. Read more about the `InterfaceFilter()` object and interface types here.

Instead of just printing the interface IP(v6) addresses, we could also print the full L2 switchport information, similar to the output of `show interface [Type0/1] switchport` by using this code in the `try` loop:

<pre class="lang:default decode:true">try:
    if_filter = InterfaceFilter(interface_type=1)
    for interface in ne.get_interface_list(if_filter):
        print interface.get_config()
</pre>

Go ahead and run that to see what the output looks like, I&#8217;ll leave it as a surprise for you.

### Print Route Table Information

Up next is an example that will gather information and print out something similar to the route table. This example has a lot more moving pieces than the previous one, so I&#8217;ll break it down after you see the code:

<pre class="lang:default decode:true ">from onep_connect import connect
from onep.routing import RIB, Routing, L3UnicastScope, L3UnicastRouteRange, L3UnicastRIBFilter, RouteRange, AppRouteTable
from onep.interfaces import NetworkPrefix

# Connect using passed in connection values
ne = connect('1.1.1.1', 'username', 'password')

try:  
    routing = Routing.get_instance(ne)

    # We need to get routes separately for IPv4 and IPv6
    # since we can't specify a Scope.AFIType of both address families 🙁
    for afi_type in (L3UnicastScope.AFIType.IPV4, L3UnicastScope.AFIType.IPV6):
        prefix = NetworkPrefix("::", 0)
        scope = L3UnicastScope("", afi_type)
        range = L3UnicastRouteRange(
                      prefix, RouteRange.RangeType.EQUAL_OR_LARGER, 0)
        filter = L3UnicastRIBFilter()
        route_list = routing.rib.get_route_list(scope, filter, range)

        for route in route_list:
            #get the first next hop only, either the interface or IP
            for next_hop in route.next_hop_list:
                next_hop = max([next_hop.address, next_hop.network_interface.name], key=len)
                break
            
            full_prefix = '%s/%s' % (route.prefix.address, route.prefix.prefix_length)
            print('%-24s  Type: %-12s AD: %-4s Metric: %-3s Next-Hop: %-20s') % \
                (full_prefix, route.OwnerType.enumval(route.owner_type), route.admin_distance, 
                    route.metric, next_hop)

finally:
    # Finally have the application disconnect from the Network Element  
    ne.disconnect()</pre>

Which gives the output:

<pre class="toolbar:2 striped:false lang:sh highlight:0 decode:true ">0.0.0.0/0                 Type: STATIC       AD: 1    Metric: 0   Next-Hop: 10.211.55.1
0.0.0.0/0                 Type: STATIC       AD: 1    Metric: 0   Next-Hop: 10.211.55.1
1.1.1.1/32                Type: LOCAL        AD: 0    Metric: 0   Next-Hop: Loopback0
10.211.55.0/24            Type: CONNECTED    AD: 0    Metric: 0   Next-Hop: GigabitEthernet1
10.211.55.200/32          Type: LOCAL        AD: 0    Metric: 0   Next-Hop: GigabitEthernet1
172.16.0.0/16             Type: STATIC       AD: 1    Metric: 0   Next-Hop: 192.168.56.10
192.168.56.0/24           Type: CONNECTED    AD: 0    Metric: 0   Next-Hop: GigabitEthernet2
192.168.56.1/32           Type: LOCAL        AD: 0    Metric: 0   Next-Hop: GigabitEthernet2
2001:56::/64              Type: CONNECTED    AD: 0    Metric: 0   Next-Hop: GigabitEthernet2
2001:56::1/128            Type: LOCAL        AD: 0    Metric: 0   Next-Hop: GigabitEthernet2
2001:100::/64             Type: CONNECTED    AD: 0    Metric: 0   Next-Hop: Loopback0
2001:100::1/128           Type: LOCAL        AD: 0    Metric: 0   Next-Hop: Loopback0
FF00::/8                  Type: LOCAL        AD: 0    Metric: 0   Next-Hop: Null0</pre>

I hope you see that there&#8217;s a lot of room for customization in this example. The exact formatting of the information can be represented in an infinite number of ways, and there&#8217;s even more attributes for the `NextHop()` and `Route()` objects that are available (<a title="Cisco OnePK - Routing Module" href="https://developer.cisco.com/media/onepk_python_api/onep.routing-module.html" target="_blank">read about them here</a>).

This example could easily be limited to either IPv4 or IPv6 routes only by only using one of the `L3UnicastScope.AFIType` values, and you can add additional filtering for route types and prefixes also. For example, to find only routes with a prefix of &#8216;192.168.0.0/24&#8217; or longer, use this for your `range` value:

<pre class="lang:default decode:true">prefix = NetworkPrefix("192.168.0.0", 24)
scope = L3UnicastScope("", L3UnicastScope.AFIType.IPV4)
range = L3UnicastRouteRange(prefix, RouteRange.RangeType.LARGER, 0)
filter = L3UnicastRIBFilter()
route_list = routing.rib.get_route_list(scope, filter, range)</pre>

Output:

<pre class="toolbar:2 striped:false lang:sh highlight:0 decode:true ">192.168.56.0/24           Type: CONNECTED    AD: 0    Metric: 0   Next-Hop: GigabitEthernet2
192.168.56.1/32           Type: LOCAL        AD: 0    Metric: 0   Next-Hop: GigabitEthernet2</pre>

\___

This is just the start! The next post will cover interacting with interfaces to shutdown and change configuration. There&#8217;s a lot to the OnePK SDK and I can&#8217;t cover it all, so start by reading the <a title="Cisco - OnePK API" href="https://developer.cisco.com/media/onepk_python_api/onep-module.html" target="_blank">API docs here</a>. Also, feel free to ask questions and I&#8217;ll do my best to answer them.