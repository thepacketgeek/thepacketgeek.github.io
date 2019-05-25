---
title: Python and onePK Offer the Power of SDN Today
date: 2014-11-12T20:10:19-07:00
author: Mat
layout: post
categories:
  - OnePK
---

Cisco announced their entry into the Software Defined Networking (SDN) arena with OnePK in early 2013. If you haven&#8217;t heard of Cisco&#8217;s OnePK yet, please read their introductions before continuing (only because they do a better job of explaining it than I do):

  * <a title="Cisco OnePK" href="http://www.cisco.com/c/en/us/products/ios-nx-os-software/onepk.html" target="_blank">Cisco&#8217;s One Platform Kit (OnePK)</a>
  * <a title="Cisco Developer - OnePK" href="https://developer.cisco.com/site/onepk/discover/overview/" target="_blank">OnePK Techincal Overview</a>

It took Cisco a while to deliver something tangible after the initial announcement, but it was certainly worth the wait. Cisco has a large amount of resources for onePK that range from videos, tutorials, code examples, SDKs in 3 languages (Java, C, python), and full API docs. I&#8217;ve been digging through these resources and there is plenty of good info to get people started with SDN.<!--more-->

I have obviously taken interest in their python SDK to see what sort of things I can automate and plan to write a series of posts as I learn. There&#8217;s a lot of potential as to what can be done with OnePK, so I will just be touching on the tip of the iceberg I&#8217;m sure. Just thinking off the top of my head, I&#8217;m imaging starting off with some scripts/apps with the following functionality:

  * Polling basic router/switch status information about interfaces, routes, CPU, memory, etc.
  * Interface statistics reports with QoS policy rules.
  * Web app that offers basic interface and ACL management for basic provisioning.
  * Monitoring tool that uses OnePK event listeners with configured actions, such as changing routes or creating alerts.

Just while I was thinking of these few ideas I quickly realized that so much more can be done. I&#8217;m happy starting with these basic ideas and I&#8217;m sure apps will get more complex as I get more familiar with OnePK and python alike. As I write scripts I will be making new posts to explain them and also keeping them in one handy <a title="GitHub - OnePK Examples" href="https://github.com/thepacketgeek/cisco-onepk-python-examples" target="_blank">GitHub repo here</a>. As I start to create bigger apps (like the web app provisioning tool) I will probably create stand-alone repos just for those, so keep an eye out for that.

### Getting Started

Here are the steps to get started developing with python and OnePK

  * Create a Cisco Developer account and download the <a title="OnePK Python SDK" href="https://developer.cisco.com/site/onepk/downloads/python/" target="_blank">OnePK SDK for Python</a>
  * Download either the <a href="http://software.cisco.com/download/release.html?mdfid=284364978&flowid=39582&softwareid=282046477&release=3.12.0S&relind=AVAILABLE&rellifecycle=ED&reltype=latest" target="_blank">CSR-1000V</a> or <a href="https://developer.cisco.com/site/onepk/downloads/all-in-one-vm/" target="_blank">OnePK All-in-One VM</a> 
      * Install instructions are available from Cisco for both
      * No, I will not provide these files to anyone, please don&#8217;t ask.
  * Watch Cisco&#8217;s <a href="https://developer.cisco.com/site/onepk/learn/self-paced-training/index.gsp" target="_blank">Self-Paced Training</a>
  * Check out the <a href="https://developer.cisco.com/site/onepk/learn/tutorials/python/" target="_blank">python OnePK tutorials</a>

### Documentation Woes

I&#8217;d like to give you a warning that Cisco&#8217;s <a title="Cisco - onep python API" href="https://developer.cisco.com/media/onepk_python_api/onep-module.html" target="_blank">main onep API docs</a> can be a pain to sift through. I don&#8217;t think the python SDK was written by someone who works with python everyday (but I could be way off-base and if so, I apologize). There are complex classes and enumerations used where much simpler methods could be used. You&#8217;ll see in examples later how many objects have to be created just to get an interface list or RIB information. I usually start off finding what I want with the examples included in the SDK and then make use of python&#8217;s `dir()` and `help()` functions to figure out what is going on with each returned object.

&#8212;

If you&#8217;re interested in harnessing the power of python to control your network elements, please follow my blog and learn with me as I dive into the world of SDN and Cisco&#8217;s OnePK.