---
title: Monitor your BGP Advertised Routes with Paramiko
date: 2015-07-07T09:37:11-07:00
author: Mat
layout: post
permalink: /monitor-your-bgp-advertised-routes-with-paramiko/
categories:
  - Networking
---
This post moves back to the basics and covers how to perform screen scraping using python and a helpful SSH library called <a href="http://www.paramiko.org" target="_blank">Paramiko</a>. Although the example below is about a specific task of checking the BGP outbound advertised routes of a device, the script can be reused to perform screen scraping for any output that the device offers. This example is for a Cisco IOS device, but it should work on many devices with a few tweaks in the commands sent and regex strings.

I've used inline comments for the explanations in the example below, but feel free to ask questions about this example or screen scraping other specific data. Here's the <a href="https://gist.github.com/thepacketgeek/9bdbc2e3818151b5c8ed" target="_blank">GitHub Gist version</a> if you want to fork and make your own changes.<!--more-->

```python
import paramiko
import re
from time import sleep

# Router connection details (could be read in from file or argparse)
peer = '172.16.2.10'
username = 'admin'
password = 'admin'

def collect_output(shell, command):
    """ Send command, and wait for output, looking for hostname as end
    of message identifier 
    """
    # Send command, but make sure there is a newline at end
    shell.send(command.strip() + '\n')
    sleep(3)

    output = ''
    while (shell.recv_ready()):
        output += shell.recv(255)

    return output

# Connect to the router using Paramiko, setup a shell to send/receive
ssh = paramiko.SSHClient()
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
try:
    ssh.connect(peer, 22, username, password, look_for_keys=False)
except (paramiko.transport.socket.error, 
        paramiko.transport.SSHException, 
        paramiko.transport.socket.timeout, 
        paramiko.auth_handler.AuthenticationException):
    print 'Error connecting to SSH on %s.' % peer
shell = ssh.invoke_shell()
shell.settimeout(3)

neighbor_output = collect_output(shell, '\nshow ip bgp summ | b Neigh\n')

# Filter and add only established BGP sessions
peer_ips = []
non_estab_states = 'neighbor|idle|active|Connect|OpenSent|OpenConfirm'
for line in neighbor_output.split('\n'):
    if bool(re.search(non_estab_states, line, re.IGNORECASE)):
        continue
    else:
        ip_regex = r'(\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})'
        peer_ips.extend(re.findall(ip_regex, line))

# Iterate through established peers and check advertised routes
for peer in peer_ips:                                                                                                   
    print 'Peer %s:' % peer
    adv_output = collect_output(shell, '\nshow ip bgp neig %s advertised\n' % peer)

    # Parse out prefixes in advertised routes
    prefix_regex = r'(\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}/\d{1,2})'
    for prefix in re.findall(prefix_regex, adv_output):                               
        print '\t%s' % prefix                                                                                           
                                                                                                                        
ssh.close() 
```

The output of will look similar to this (depending on your actual peers and advertised prefixes of course):

```
Peer 172.16.2.10:
        100.10.10.0/24
        100.20.20.0/24
        200.0.0.0/16
Peer 172.16.2.20:
        100.10.10.0/24
        100.20.20.0/24
        200.0.0.0/16
```


I hope you found this interesting and helpful!