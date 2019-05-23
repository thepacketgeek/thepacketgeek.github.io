---
id: 50
title: Sniff Packets and View them Live in the Cloud!
date: 2013-10-09T19:35:42-07:00
author: Mat
layout: post
guid: http://thepacketgeek.com/?p=50
permalink: /sniff-packets-and-view-them-in-the-cloud/
categories:
  - Coding
  - Networking
---
So, I just uploaded two GitHub repos that I&#8217;m really excited about: <a title="Meteorshark" href="https://www.github.com/thepacketgeek/meteorshark" target="_blank">Meteorshark</a> and <a title="Scapy-to-API" href="https://www.github.com/thepacketgeek/scapy-to-api" target="_blank">Scapy-to-API</a>

<p style="text-align: center;">
  <a href="http://thepacketgeek.com/wp-content/uploads/2013/10/meteorshark1.png"><img class=" wp-image-52 aligncenter" alt="meteorshark1" src="//thepacketgeek.com/wp-content/uploads/2013/10/meteorshark1-1024x501.png" width="584" height="285" srcset="https://thepacketgeek.com/wp-content/uploads/2013/10/meteorshark1-1024x501.png 1024w, https://thepacketgeek.com/wp-content/uploads/2013/10/meteorshark1-300x146.png 300w, https://thepacketgeek.com/wp-content/uploads/2013/10/meteorshark1-500x244.png 500w, https://thepacketgeek.com/wp-content/uploads/2013/10/meteorshark1.png 1238w" sizes="(max-width: 584px) 100vw, 584px" /></a>
</p>

<!--more-->This started out as an idea for a 

<a title="Scapy" href="http://www.secdev.org/projects/scapy/" target="_blank">Scapy</a>&nbsp;presentation/demo I&#8217;ll be doing for the Sonora Developers Meet-up group (more Scapy articles and presentation details coming soon). I wanted a way to sniff some packets on my local machine and present those to the attendees. &nbsp;Since this is a general developer&#8217;s group I&#8217;m sure there will be people not familiar with the verbose output of tcpdump or Wireshark. Meteorshark is a way to see the most relevant packet info at a glance, while still being able to see the full packet details if desired. The companion to Meteorshark, Scapy-to-API, is what runs on my local machine, sniffing packets and uploading them to the Meteorshark API to dump into the database.

This is definitely an on-going work in progress as I have several presentation, performance, and possibly security issues to work out. But I did it and I&#8217;m really proud. I will definitely be writing about Meteor as it definitely made this project much easier to prototype and I&#8217;d probably still be struggling with other frameworks to this day.

So, check out the projects, try them out, and watch the repo so you can see the features come rolling in!