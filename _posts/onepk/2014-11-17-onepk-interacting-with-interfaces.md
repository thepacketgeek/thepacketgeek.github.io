---
id: 424
title: 'OnePK &#8211; Interacting with Interfaces'
date: 2014-11-17T07:48:39-07:00
author: Mat
layout: post
guid: http://thepacketgeek.com/?p=424
permalink: /onepk-interacting-with-interfaces/
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
  This entry is part 4 of 5 in the series <a href="https://thepacketgeek.com/series/cisco-onepk/" class="series-24" title="Getting started with Cisco OnePK">Getting started with Cisco OnePK</a>
</div>

Getting information from your network devices is really helpful, but actually change device configurations is even more helpful! This post will have a few examples of how to do just that with scripts that will shutdown a specified interface and change an interface IP address. This is where the fun begins, so strap into your chairs and get ready for some network automation!<!--more-->

### Shutting down an Interface

This first example&#8217;s goal is to shut down an interface programmatically. In order to specify an interface, we&#8217;ll search the interfaces for a passed in IP address.

<pre class="lang:default decode:true">from onep_connect import connect
from onep.interfaces import InterfaceFilter

# Connect to the network element
# (will raise a ValueError if bad IP address or credentials)
ne = connect('1.1.1.1', 'admin', 'admin')

try:  
    #Create Interface Filter and find interface by IP
    if_filter = InterfaceFilter(interface_type=1)
    for interface in ne.get_interface_list(if_filter):
        if '192.168.1.10' in interface.get_address_list():
            try:
                interface.shut_down(1)
                print '%s has been shutdown.' % interface.name

finally:
    # Finally have the application disconnect from the Network Element  
    ne.disconnect()</pre>

When we run this script from the command line, this is the output:

<pre class="toolbar:2 striped:false lang:sh highlight:0 decode:true">$ python print_all_interface_IPs.py 10.211.55.200 admin admin 1.1.1.1 1
Loopback0 has been shutdown.</pre>

Ok, let&#8217;s break this down now. We use really similar code to an example in the previous post that will get the IP addresses for each interface. We iterate through each interface looking for the IP addresses specified, and apply the `.shut_down()` method to it. You can see in the <a title="GitHub - Shutting down an Interface" href="https://github.com/thepacketgeek/cisco-onepk-python-examples/blob/master/scripts/shutdown_interface_by_ip.py" target="_blank">full example here</a>, if a value of 1 (shutdown) or 0 (enable) is passed into the \`.shut_down()\` method, that value is passed to the method, otherwise we just shutdown the interface with a value of 1.

### Changing an Interface IP Address

If you&#8217;re going through an IP address range migration this next example could be very valuable to shave hours off the update process. Let&#8217;s look at what it takes to change and Interface IP address programmatically:

<pre class="lang:default decode:true ">from onep_connect import connect
from onep.core.util import OnepConstants
from onep.core.util import HostIpCheck

# Connect using IOS device connection details
ne = connect('1.1.1.1', 'admin', 'admin')

try:  
    #set IP address and prefix
    ip_address = '192.168.1.10'
    ip_prefix = 24
    
    # Test for IPv4 or IPv6 address and assign accordingly
    if HostIpCheck(ip_address).is_ipv4():
        scope_type = OnepConstants.OnepAddressScopeType.ONEP_ADDRESS_IPv4_PRIMARY
    elif HostIpCheck(ip_address).is_ipv6():
        scope_type = OnepConstants.OnepAddressScopeType.ONEP_ADDRESS_IPv6_ALL
    else:
        raise ValueError('%s is not a valid IP address' % ip_address)
        
    #get interface by name
    interface_name = 'Fa0/1'
    try:
        iface = ne.get_interface_by_name(inteface_name)
    except:
        raise ValueError('The \'%s\' interface does not exist on this Network Element.' % interface_name)

    iface.set_address(1, scope_type, ip_address, ip_prefix)

    print 'The %s interface has been configured with the address: %s/%s' % (interface_name, ip_address, ip_prefix)

finally:
    # Finally have the application disconnect from the Network Element  
    ne.disconnect()</pre>

It looks like there&#8217;s a lot going on, but there are really just a few simple steps:

  * Connect to the IOS device
  * Set the IP address and prefix values (this can be done through user input as well).
  * Test the IP address to make sure it is valid.
  * Set the address on the interface we specified by name.
  * Print the output and disconnect.

The only new object here is the \`OnepAddressScopeType\`, you can read about the possible scope types <a title="OnepAddressScopeType" href="https://developer.cisco.com/media/onepk_python_api/onep.core.util.OnepConstants.OnepConstants-class.html#OnepAddressScopeType" target="_blank">here</a>. Also, you can see more options when using the \`.set_address()\` method of the NetworkInterface object <a title="OnePK - .set_address() method" href="https://developer.cisco.com/media/onepk_python_api/onep.interfaces.NetworkInterface.NetworkInterface-class.html#set_address" target="_blank">here</a>.

To see how this example can be made into a CLI script which takes all the values in as arguments, check out the full example in my <a title="GitHub - Setting IP address" href="https://github.com/thepacketgeek/cisco-onepk-python-examples/blob/master/scripts/set_interface_ip_address.py" target="_blank">OnePK GitHub repo here</a>.

### Saving the Network Element Configuration

Being able to make these changes via script is great for bulk changes, but remember that a change is only saved in running_configuration until it&#8217;s written to the NVRAM of the IOS device. These changes above would be lost with a reload or power off. Luckily though, the OnePK library includes a \`VtyHelper\` class to add some config saving abilities to our script:

<pre class="lang:default decode:true">from one_connect import connect
# This is our VtyHelper class
from onep.vty.util import VtyHelper

# Connect to our network element
ne = connect('1.1.1.1', 'admin', 'admin')

###  Make config changes (interface IPs, shutdown interfaces)

# Now we're ready to save the config
ne_vty = VtyHelper(ne)
ne_vty.commit()
# Save confirmed with output: '\r\nBuilding configuration...\r\n[OK]'

# Saved and ready to disconnect
ne.disconnect()</pre>

The Vty module in OnePK has plenty of other options for running commands and receiving output via the CLI if the other OnePK classes don&#8217;t offer what you need. I&#8217;ll go more in depth in a later post.

&#8212;

This was just an introduction to making configuration changes on an IOS device. The next post will cover more interface config changes like applying a VLAN or adding ACLs. I know, I&#8217;m just as excited as you are! Thanks for reading and check back soon.