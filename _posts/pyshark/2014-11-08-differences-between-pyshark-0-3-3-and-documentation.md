---
title: Differences between PyShark 0.3.3 and Documentation
date: 2014-11-08T16:44:28-07:00
author: Mat
layout: post
categories:
  - PyShark
---
While working with Pyshark I&#8217;ve found that some of the documentation doesn&#8217;t quite line up so I&#8217;m writing this post to help people that might run into the same situation. The intro doc is found <a title="PyShark" href="http://kiminewt.github.io/pyshark/" target="_blank">here</a> and I&#8217;ll be comparing it to what actually happens when using the newest version (0.3.3) of Pyshark.

<!--more-->The first thing I noticed is that it&#8217;s difficult to get a basic count of the packets in the capture object. The \_\_repr\_\_ string doesn&#8217;t include a packet count when reading from a file (only available when using LiveCapture):

<img class="aligncenter wp-image-341 size-large" src="//thepacketgeek.com/wp-content/uploads/2014/11/pyshark1-1024x446.png" alt="pyshark1" width="650" height="283" srcset="https://thepacketgeek.com/wp-content/uploads/2014/11/pyshark1-1024x446.png 1024w, https://thepacketgeek.com/wp-content/uploads/2014/11/pyshark1-300x130.png 300w, https://thepacketgeek.com/wp-content/uploads/2014/11/pyshark1.png 1448w" sizes="(max-width: 650px) 100vw, 650px" /> 

<pre class="lang:default decode:true ">&gt;&gt;&gt; cap = pyshark.FileCapture('test.pcap')
&gt;&gt;&gt; cap
   &lt;FileCapture test.pcap&gt;
&gt;&gt;&gt;</pre>

And checking the&nbsp;`len()`&nbsp;of cap tells me that the packets are only read in when requested (lazy fetching):

<pre class="lang:default decode:true ">&gt;&gt;&gt; cap[10]
   &lt;UDP/HTTP Packet&gt;
&gt;&gt;&gt; len(cap)
   11
&gt;&gt;&gt;</pre>

But, I think this was changed in order to improve performance since I see references to a&nbsp;`lazy`&nbsp;parameter in earlier commits, and there is also a current option to not keep the packets in memory (in the cap._packets list). When this option is used, you can only iterate through the packets and not reference them by index:

<img class="aligncenter size-large wp-image-343" src="//thepacketgeek.com/wp-content/uploads/2014/11/pyshark2-1024x248.png" alt="pyshark2" width="650" height="157" srcset="https://thepacketgeek.com/wp-content/uploads/2014/11/pyshark2-1024x248.png 1024w, https://thepacketgeek.com/wp-content/uploads/2014/11/pyshark2-300x72.png 300w, https://thepacketgeek.com/wp-content/uploads/2014/11/pyshark2.png 1524w" sizes="(max-width: 650px) 100vw, 650px" /> 

<pre class="lang:default decode:true ">&gt;&gt;&gt; cap = pyshark.FileCapture('test.pcap', keep_packets=False)
&gt;&gt;&gt; len(cap)
   0
&gt;&gt;&gt; cap[10]
... (truncated)
    NotImplementedError: Cannot use getitem if packets are not kept
&gt;&gt;&gt; for pkt in cap:
....:     print pkt.highest_layer
....:
HTTP
HTTP
HTTP
... (truncated)</pre>

&nbsp;