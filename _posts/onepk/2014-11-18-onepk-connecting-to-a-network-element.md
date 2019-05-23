---
id: 406
title: 'OnePK &#8211; Connecting to a Network Element'
date: 2014-11-18T08:20:14-07:00
author: Mat
layout: post
guid: http://thepacketgeek.com/?p=406
permalink: /onepk-connecting-to-a-network-element/
categories:
  - Coding
  - Networking
tags:
  - CSR-1000V
  - OnePK
  - Python
series:
  - Getting started with Cisco OnePK
---
<div class="seriesmeta">
  This entry is part 2 of 5 in the series <a href="https://thepacketgeek.com/series/cisco-onepk/" class="series-24" title="Getting started with Cisco OnePK">Getting started with Cisco OnePK</a>
</div>

The first step in managing your network with Cisco&#8217;s OnePK is learning how to connect to a switch or router, what Cisco calls a Network Element. In the early OnePK days this was a very straightforward task using vanilla TCP but in the newest version of OnePK (1.3) and IOS (15.4), unencrypted communications were disabled and we are forced to use TLS. This makes sense from a network security point-of-view; it just makes it a little more difficult to get started.

Fortunately, amongst Cisco&#8217;s vast resources I found a document that helps outline a process that makes it easier to use TLS between our OnePK apps and Cisco IOS devices. The guide uses a technique called TLS pinning which allows our OnePK app to bypass certificates but still encrypt communications via TLS. Read more about this technique here: <a title="Cisco - TLS Pinning Guide" href="https://communities.cisco.com/thread/44820" target="_blank">Cisco &#8211; TLS Pinning Guide</a>. (Please note that this should not be used for production as it does not verify the endpoints. Certificates should be used for TLS in a production network.)

<!--more-->

### Configuring the IOS device for OnePK

Before the Cisco IOS device will accept OnePK connections, you will need to add at least one username (privilege level 15) and enable the `onep` service. The `disable-remotecert-validation` keyword is required for the TLS pinning mentioned above:

<pre class="toolbar:2 striped:false lang:sh highlight:0 decode:true ">HostName(config)# username [username] privilege-level 15 secret [secret]
HostName(config)# onep
HostName(config-onep)#transport type tls disable-remotecert-validation</pre>

### Using TSL Pinning in a python Script

With the TLS Pinning Guide above, our initial connect script will look something like this:

<pre class="lang:default decode:true"># Import the onePK Libraries  
from onep.element.NetworkElement import NetworkElement  
from onep.element.SessionConfig import SessionConfig
from onep.core.util import tlspinning  
from onep.interfaces import InterfaceFilter 
  
# TLS Connection (This is the TLS Pinning Handler)  
class PinningHandler(tlspinning.TLSUnverifiedElementHandler):  
    def __init__(self, pinning_file):  
        self.pinning_file = pinning_file  
    def handle_verify(self, host, hashtype, finger_print, changed):  
        return tlspinning.DecisionType.ACCEPT_ONCE  
  
# Setup a connection config with TSL pinning handler
config = SessionConfig(None)  
config.set_tls_pinning('', PinningHandler(''))  
config.transportMode = SessionConfig.SessionTransportMode.TLS  

# Connection to my onePK enabled Network Element  
ne = NetworkElement('1.1.1.1', 'App_Name')  
ne.connect('username', 'password', config)  

# Print the information of the Network Element  
print ne

# Finally have the application disconnect from the Network Element  
ne.disconnect()</pre>

This script will connect to the network device 1.1.1.1 and print out details about the Network Element, with similar output to what&#8217;s below (I&#8217;m using a CSR-1000v hosted locally on my laptop):

<pre class="toolbar:2 striped:false lang:sh highlight:0 decode:true">NetworkElement [ 1.1.1.1 ]
	Product ID   : CSR1000V
	Processor    : VXE
	Serial No    : 9KAAAAAAAA
	sysName      : CSR1
	sysUpTime    : 42135
	sysDescr     : Cisco IOS Software, CSR1000V Software (X86_64_LINUX_IOSD-UNIVERSALK9-M), Version 15.4(2)S, RELEASE SOFTWARE (fc2)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2014 by Cisco Systems, Inc.
Compiled Wed 26-Mar-14 21:09 by mcpre</pre>

###  Simplifying the OnePK Connection

With the OnePK connection code in a python script/app, it isn&#8217;t too much of a pain to type this in once, but what about using python in interactive mode for testing and OnePK discovery?  I&#8217;ve created a single python file that you can use as a module to quickly connect to IOS devices for testing so you don&#8217;t have to copy paste this code all the time. If you download the <a title="Onep_connect.py" href="https://github.com/thepacketgeek/cisco-onepk-python-examples/blob/master/scripts/onep_connect.py" target="_blank"><code>onep_connect.py</code></a> file from my <a title="GitHub - Cisco OnePk Examples" href="https://github.com/thepacketgeek/cisco-onepk-python-examples" target="_blank">OnePK example GitHub repo here</a>, you can save that to the local folder with your scripts or where you&#8217;re running python/iPython from. With the file ready locally, the script above becomes as straightforward as this example:

<pre class="lang:default decode:true  ">from onep_connect import connect

ne = connect('1.1.1.1', 'username', 'password')
print ne
ne.disconnect()</pre>

Much better, right?

### Disconnecting from the Network Element

The onep connection to the network element is stateful and the IOS device will keep track of the connection. If you forget to run the `.disconnect()` method when you are finished with the connection, you may see the following error if you try to connect again:

<pre class="toolbar:2 lang:default decode:true ">OnepDuplicateElementException: Error occurred in the operation.  Duplicate IP address.</pre>

If this happens, don&#8217;t panic. You can login to the CLI of the IOS device and execute the \`onep stop session all\` to clear the onep sessions, or specify a certain application with the \`onep stop session [app_name]\` command.

&#8212;

Now that we know how to connect to a Cisco IOS device with OnePK, tune in for more articles about how to get information from your network devices and eventually, how to interact with them and make config changes.