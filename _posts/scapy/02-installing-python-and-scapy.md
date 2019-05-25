---
title: 'Scapy p.02 &#8211; Installing Python and Scapy'
date: 2018-10-29T16:59:58-07:00
author: Mat
layout: post
categories:
  - scapy
---
#### Installing Python

Scapy was originally written for Python 2, but since the 2.4 release (March 2018), you can now use Scapy with Python 3.4+! I will prefer Python 3 in examples but will also include notes about big differences between each python version and Scapy if they exist.

If you&#8217;re using a Mac or running some version of *nix you probably already have Python 2 (and maybe even Python 3) installed. To check, open a terminal and type `python3` or `python`. You should see something like this:

<pre class="lang:default decode:true">localhost:~ packetgeek$ python3
Python 3.7.2 (v3.7.2:9a3ffc0492, Dec 24 2018, 02:44:43)
[Clang 6.0 (clang-600.0.57)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
&gt;&gt;&gt;</pre>

<pre class="lang:default decode:true">localhost:~ packetgeek$ python
Python 2.7.10 (default, Aug 17 2018, 19:45:58)
[GCC 4.2.1 Compatible Apple LLVM 10.0.0 (clang-1000.0.42)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
&gt;&gt;&gt;</pre>

If you are running Windows or for some other reason do not have Python installed already, go to the <a href="http://python.org/download/" target="_blank" rel="noopener noreferrer">Python download page</a> and grab the installer for your platform.

* * *

#### Installing Scapy

There are multiple ways to install Scapy depending on your platform. Check out the <a href="https://scapy.readthedocs.io/en/latest/installation.html#installing-scapy-v2-x" target="_blank" rel="noopener noreferrer">Scapy installation guides</a> to find instructions and installer packages relevant to your platform. Once Scapy is installed, you should be able to run it from the terminal, just like we did with Python, and get something that looks like this:

<pre class="lang:default decode:true">localhost:~ packetgeek$ scapy

                     aSPY//YASa
             apyyyyCY//////////YCa       |
            sY//////YSpcs  scpCY//Pp     | Welcome to Scapy
 ayp ayyyyyyySCP//Pp           syY//C    | Version 2.4.2
 AYAsAYYYYYYYY///Ps              cY//S   |
         pCCCCY//p          cSSps y//Y   | https://github.com/secdev/scapy
         SPPPP///a          pP///AC//Y   |
              A//A            cyP////C   | Have fun!
              p///Ac            sC///a   |
              P////YCpc           A//A   | We are in France, we say Skappee.
       scccccp///pSP///p          p//Y   | OK? Merci.
      sY/////////y  caa           S//P   |             -- Sebastien Chabal
       cayCyayP//Ya              pY/Ya   |
        sY/PsY////YCc          aC//Yp
         sc  sccaCY//PCypaapyCP//YSs
                  spCPY//////YPSps
                       ccaacs
                                       using IPython 7.2.0
&gt;&gt;&gt;</pre>

I highly recommend install IPython in your scapy environment as it makes interactive mode much more enjoyable!

<pre class="lang:default decode:true ">pip3 install ipython</pre>

<!--more-->

* * *

#### Scapy and Network Interfaces

If you have multiple network interfaces on your computer, you might have to double check which interface Scapy will use by default. Run `scapy` from the terminal and run the `conf` command. See what interface Scapy will use by default by looking at the `iface` value:

<pre class="lang:default decode:true">localhost:~ packetgeek$ scapy
Welcome to Scapy (2.2.0)
&gt;&gt;&gt; conf.iface
'en0'</pre>

<p class="caption">
  Scapy on my computer is defaulted to my en0 (Wifi) interface
</p>

If the default interface is not the one you will use, you can change the value like this:

<pre class="lang:default decode:true">&gt;&gt;&gt; conf.iface="en3"</pre>

<p class="caption">
  *Instead of en3, use the interface you want to be your default
</p>

If you are constantly switching back and forth between interfaces, you can specify the interface to use when you run Scapy commands. Here are some Scapy functions and how you might use the `iface` argument. You&#8217;ll learn more about these functions and other arguments soon.

<pre class="lang:default decode:true">&gt;&gt;&gt; sniff(count=10, iface="en3")
&gt;&gt;&gt; send(pkt, iface="en3")</pre>

#### Root Permissions

For some of the Scapy functions dealing with sending traffic, you will need to be able to run Scapy as root. For example, on a Mac/Linux computer you can run interactive mode with root permissions by running `sudo scapy` from the CLI. Also, you can run the Python scripts with Scapy send functions as sudo like this:

    $ sudo python3 script.py

If you ever receive a Python error about not having the correct permissions or something not be allowed, try running Scapy with root permissions.