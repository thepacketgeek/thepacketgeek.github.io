+++
title = "Scapy p.09"
description = "Scapy and DNS"
#date = 2013-10-29
date = 2019-05-11
author = "Mat"
weight = 92
in_search_index = true
aliases = ["/scapy-p-09-scapy-and-dns/"]
[taxonomies]
tags = ["scapy", "python"]
+++

We've been able to work with Ethernet, ARP, IP, ICMP, and TCP pretty easily so far thanks to Scapy's built in protocol support. Next on our list of protocols to work with are UDP and DNS.

#### DNS Request and Response

Using the `sr1()` function, we can craft a DNS request and capture the returned DNS response. <!-- more -->Since DNS runs over IP and UDP, we will need to use those in our packet:

```python
#! /usr/bin/env python3

from scapy.all import DNS, DNSQR, IP, sr1, UDP

dns_req = IP(dst='8.8.8.8')/
    UDP(dport=53)/
    DNS(rd=1, qd=DNSQR(qname='www.thepacketgeek.com'))
answer = sr1(dns_req, verbose=0)

print(answer[DNS].summary())
```

Console Output:
```
Begin emission:
..Finished to send 1 packets.
..*
Received 5 packets, got 1 answers, remaining 0 packets
DNS Ans "198.71.55.197"
```

> The DNS layer summary is printed showing the IP address of the hostname requested

Without too much work we were able to write a short script to query some DNS name to IP address resolutions.&nbsp;Just think about what this means; we could read in a list of hostnames to resolve and send the IP addresses off to some function to do some more tests such as ping or TCP scans.

#### DNS Forwarding and Spoofing

Ok, so sending a DNS query was fun, but let's build on that. How about hand building a DNS service that can handle DNS forwarding, but with the added functionality of handing out a custom IP address for a certain domain name. This is similar to how the DNS server part of the <a href="https://github.com/iBaa/PlexConnect" target="_blank" rel="noopener noreferrer">PlexConnect</a> utility works to hijack some communications from the AppleTV. This is going to jump up in complexity quite a bit from our previous example but the process is still pretty simple:

  * Sniff with Scapy to listen for incoming DNS requests 
      * Filtering for UDP port 53 destined to the server's IP address
  * If the request is for our special domain name, send a spoofed DNS response 
      * Swap source/dest UDP ports and IP addresses
      * Match DNS request ID
  * Otherwise, make a new DNS request and send the response back to the requesting host 
      * Make a new request, and save the DNS Response
      * Send a response back to the client matching the same fields as above

We're doing a lot of field replacements, especially on the handcrafted spoof response. All I did to figure out all those fields is capture a DNS response from a request I made and used the `show()` function to figure out what fields are expected in the DNS response.

```python
#! /usr/bin/env python3

from scapy.all import DNS, DNSQR, DNSRR, IP, send, sniff, sr1, UDP

IFACE = "lo0"   # Or your default interface
DNS_SERVER_IP = "127.0.0.1"  # Your local IP

BPF_FILTER = f"udp port 53 and ip dst {DNS_SERVER_IP}"


def dns_responder(local_ip: str):

    def forward_dns(orig_pkt: IP):
        print(f"Forwarding: {orig_pkt[DNSQR].qname}")
        response = sr1(
            IP(dst='8.8.8.8')/
                UDP(sport=orig_pkt[UDP].sport)/
                DNS(rd=1, id=orig_pkt[DNS].id, qd=DNSQR(qname=orig_pkt[DNSQR].qname)),
            verbose=0,
        )
        resp_pkt = IP(dst=orig_pkt[IP].src, src=DNS_SERVER_IP)/UDP(dport=orig_pkt[UDP].sport)/DNS()
        resp_pkt[DNS] = response[DNS]
        send(resp_pkt, verbose=0)
        return f"Responding to {orig_pkt[IP].src}"

    def get_response(pkt: IP):
        if (
            DNS in pkt and
            pkt[DNS].opcode == 0 and
            pkt[DNS].ancount == 0
        ):
            if "trailers.apple.com" in str(pkt["DNS Question Record"].qname):
                spf_resp = IP(dst=pkt[IP].src)/UDP(dport=pkt[UDP].sport, sport=53)/DNS(id=pkt[DNS].id,ancount=1,an=DNSRR(rrname=pkt[DNSQR].qname, rdata=local_ip)/DNSRR(rrname="trailers.apple.com",rdata=local_ip))
                send(spf_resp, verbose=0, iface=IFACE)
                return f"Spoofed DNS Response Sent: {pkt[IP].src}"

            else:
                # make DNS query, capturing the answer and send the answer
                return forward_dns(pkt)

    return get_response

sniff(filter=BPF_FILTER, prn=dns_responder(DNS_SERVER_IP), iface=IFACE)
```


With this running on a host, I used the DNS `dig` utility to make some DNS requests:

```sh
localhost:~ packetgeek$ dig @172.16.20.40 www.thepacketgeek.com

; <<>> DiG 9.8.5-P1 <<>> @172.16.20.40 www.thepacketgeek.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 29980
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;www.thepacketgeek.com.   IN  A

;; ANSWER SECTION:
www.thepacketgeek.com.  20411 IN  A 198.71.55.197

;; Query time: 90 msec
;; SERVER: 172.16.20.40#53(172.16.20.40)
;; WHEN: Thu Oct 10 19:39:38 PDT 2013
;; MSG SIZE  rcvd: 76

localhost:~ packetgeek$ dig @172.16.20.40 trailers.apple.com

; <<>> DiG 9.8.5-P1 <<>> @172.16.20.40 trailers.apple.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12688
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0
;; WARNING: Messages has 34 extra bytes at end

;; QUESTION SECTION:
;trailers.apple.com.    IN  A

;; ANSWER SECTION:
trailers.apple.com. 20  IN  A 172.16.20.40

;; Query time: 561 msec
;; SERVER: 172.16.20.40#53(172.16.20.40)
;; WHEN: Thu Oct 10 19:39:45 PDT 2013
;; MSG SIZE  rcvd: 104<
```


This example uses a lot of Python, so if you're not familiar with that take some time to look at the code and look up anything you don't know in the <a href="https://docs.python.org/3.8/" target="_blank" rel="noopener noreferrer">Python Docs</a>.