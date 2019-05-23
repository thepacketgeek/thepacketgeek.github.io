---
id: 12
title: Projects
date: 2013-09-21T04:21:17-07:00
author: Mat
layout: page
guid: http://thepacketgeek.com/?page_id=12
---
This list will hopefully grow in the future but right now here is what I&#8217;m working on:

## ExaBGPmon <a class="project-link" title="Github - sanicap" href="https://github.com/thepacketgeek/ExaBGPmon" target="_blank">github</a>

A HTTP GUI frontend to the glorious ExaBGP tool. This project makes it easier to deploy, configure, and monitor ExaBGP and its BGP peerings, along with controlling route advertisements.

## Sanicap <a class="project-link" title="Github - sanicap" href="https://github.com/thepacketgeek/sanicap" target="_blank">github</a>

Python based utility that sanitizes pcap files. Options for sequential addressing, address masks and sequential starting addresses. Used to allow for cloud-based sanitation of pcap files in my Cloud-pcap project.

## Cloud-PCAP <a class="project-link" title="Github - cloud-pcap" href="https://github.com/thepacketgeek/cloud-pcap" target="_blank">github</a>

Web based .pcap/.pcapng storage and lightweight analysis using python + Flask. A tribute to cloudshark.org, but on a charmingly pathetic level. This project uses <a title="Pyshark" href="http://kiminewt.github.io/pyshark/" target="_blank">Pyshark</a> for packet decoding.

## Cisco-OnePk-Python-Examples <a class="project-link" title="cisco-onepk-python-examples" href="https://github.com/thepacketgeek/cisco-onepk-python-examples" target="_blank">github</a>

A repo that accompanies several posts on OnePK. Hopefully the examples included help others get started with developing their own Cisco OnePk projects.

## Meteorshark <a class="project-link" title="GitHub - meteorshark" href="https://github.com/thepacketgeek/meteorshark" target="_blank">github</a>

This is an idea to easily show packet information in demos instead of getting lost in the verbose tcpdump output or overwhelming Wireshark interface. This will be an ongoing project for me and should mature a lot as my dev and packet skills improve. This is a companion app, to be use with some sort of client side sniffer, such as my next project: scapy-to-api

## Scapy-to-API <a class="project-link" title="GitHub - scapy-to-api" href="https://github.com/thepacketgeek/scapy-to-api" target="_blank">github</a>

Intended for use with Meteorshark, this is a Python script that uses scapy to sniff packets and upload them to a DB via API.

## Cisco-Cert-Unofficial-API <a class="project-link" title="GitHub - cisco-cert-unofficial-api" href="https://github.com/thepacketgeek/Cisco-Cert-Unofficial-API" target="_blank">github</a> <a class="project-link" title="NPM cisco-cert-api-server" href="https://npmjs.org/package/cisco-cert-api-server" target="_blank">npm</a>

This is a middleware API to allow for programmatically verifying Cisco certifications. The intended purpose is to add the ability to add Cisco cert info to profiles at <a title="CoderBits" href="http://www.coderbits.com" target="_blank">CoderBits</a>.