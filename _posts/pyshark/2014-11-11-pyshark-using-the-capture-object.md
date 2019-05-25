---
title: 'PyShark &#8211; Using the capture Object'
date: 2014-11-11T09:15:34-07:00
author: Mat
layout: post
categories:
  - PyShark
---

Before we get started you should read a few things in this post about the <a title="Differences between PyShark 0.3.3 and Documentation" href="http://thepacketgeek.com/differences-between-pyshark-0-3-3-and-documentation/" target="_blank">differences here</a> between the current version of PyShark (0.3.3) and the documentation on the website. Everything I cover in this post will be things I&#8217;ve tested and confirmed work in the current version.

Now that we know how to use the FileCapture and LiveCapture modules to capture some packets, let&#8217;s see what options we have with the returned capture object (truncated list for brevity):

<pre class="lang:default decode:true ">dir(cap)
Out[3]:
['apply_on_packets',
 'close',
 'current_packet',
 'display_filter',
 'encryption',
 'input_filename',
 'next',
 'next_packet']</pre>

<!--more-->These are the methods/attributes that I feel are actually useful, most of the other ones are used for debugging or internally for the capture process. The 

`display_filter, encryption, input_filename` attributes are used for displaying parameters passed into  FileCapture or LiveCapture.

The real magic here is the `apply_on_packets()` and `next()` methods. Iteration (via `for` loop) is available because of the `next()` method, and `apply_on_packets()` is another way to iterate through the packets, passing in a function to apply to each packet:

<pre class="lang:default decode:true">&gt;&gt;&gt; cap = pyshark.FileCapture('test.pcap', keep_packets=False)
&gt;&gt;&gt; def print_highest_layer(pkt)
...: print pkt.highest_layer
&gt;&gt;&gt; cap.apply_on_packets(print_highest_layer)
HTTP
HTTP
HTTP
HTTP
HTTP
... (truncated)</pre>

This can also be used for things other than printing, such as adding the packets to a list for counting or other processing. Here&#8217;s a script that will append all the packets to a list and print the count:

<pre class="lang:default decode:true">import pyshark

def get_capture_count():
    p = pyshark.FileCapture('test.cap.pcap', keep_packets=False)

    count = []
    def counter(*args):
        count.append(args[0])

    p.apply_on_packets(counter, timeout=100000)

    return len(count)

print get_capture_count()</pre>

&#8212;

Check out the [next PyShark article](http://thepacketgeek.com/pyshark-using-the-packet-object/ "PyShark – Using the packet Object") that covers the methods and attributes of the PyShark `packet` object.