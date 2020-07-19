+++
title = "Using service health checks to automate ExaBGP"
date = 2015-05-21
author = "Mat"
in_search_index = true
weight = 90

aliases = ["/using-service-health-checks-to-automate-exabgp/"]

[taxonomies]
tags = ["exabgp", "python"]
+++

If you haven't read the intro post to this series, [Getting Started with ExaBGP](@/exabgp/getting-started/index.md), check that article out first as this post will be expanding on the previous example to show how to use python for more automated interaction with ExaBGP.

<!-- more -->
The first example I'll cover is using a basic health check test to determine if a route should be announced or withdrawn from BGP. Here is our python script `healthcheck.py` with comments inline:

```python
#!/usr/bin/env python3

import socket
from sys import stdout
from time import sleep


def is_alive(address, port):
    """ This is a function that will test TCP connectivity of a given
    address and port. If a domain name is passed in instead of an address,
    the socket.connect() method will resolve.
 
    address (str): An IP address or FQDN of a host
    port (int): TCP destination port to use
 
    returns (bool): True if alive, False if not
    """
 
    # Create a socket object to connect with
    s = socket.socket()
    
    # Now try connecting, passing in a tuple with address & port
    try:
        s.connect((address, port))
        return True
    except socket.error:
        return False
    finally:
        s.close()

while True:
    if is_alive('thepacketgeek.com', 80):
        stdout.write('announce route 100.10.10.0/24 next-hop self' + '\n')
        stdout.flush()
    else:
        stdout.write('withdraw route 100.10.10.0/24 next-hop self' + '\n')
        stdout.flush()
    sleep(10)
```

Let's update our ExaBGP's `conf.ini` to run this python script:

```ini
process healthcheck {
    run /path/to/python3 /path/to/healthcheck.py;
    encoder json;
}

neighbor 172.16.2.10 {
    router-id 172.16.2.1;
    local-address 172.16.2.1;
    local-as 65000;
    peer-as 65000;

    api {
        processes [healthcheck];
    }
}
```

Now when we run ExaBGP with `$ exabgp conf.ini`, the health check python script will also run and push either 'announce' or 'withdraw' commands depending on the service availability. It doesn't matter that we're sending BGP UPDATE messages with the same route over and over again, but we may want to increase the delay if it's causing high CPU on the device. Here's what we see in the ExaBGP output about every 10 seconds:

```sh
... | INFO | 1095 | processes | Command from process healthcheck : announce route 100.10.10.0/24 next-hop self
... | INFO | 1095 | reactor | Route added to neighbor 172.16.2.128 local-ip 172.16.2.1 local-as 65000 peer-as 65000 router-id 172.16.2.1 family-allowed in-open : 100.10.10.0/24 next-hop 172.16.2.1
... | INFO | 1095 | reactor | Performing dynamic route update
... | INFO | 1095 | reactor | Updated peers dynamic routes successfully
```

Very cool, right? Ok, but now what if we block/kill the TCP check?

```sh
... | INFO     | 1095   | processes     | Command from process healthcheck : withdraw route 100.10.10.0/24 next-hop self
... | INFO     | 1095   | reactor       | Route removed : 100.10.10.0/24 next-hop 172.16.2.1
... | INFO     | 1095   | reactor       | Performing dynamic route update
... | INFO     | 1095   | reactor       | Updated peers dynamic routes successfully
```

Boom, routes removed. That's fantastic. What if we want to check multiple end hosts and automate different prefix advertisements? Well, that could entail an entire application to manage and monitor the end host and prefix objects, but here's a quick and dirty modification of our `healthcheck.py` script:

```python
#!/usr/bin/env python3

import socket
from collections import namedtuple
from sys import stdout
from time import sleep

 
def is_alive(address, port):
    """ same as above, removed for brevity """

# Add namedtuple object for easy reference below
TrackedObject = namedtuple('EndHost', ['address','port','prefix', 'nexthop'])


# Make a list of these tracked objects
tracked_objects = [
    TrackedObject('thepacketgeek.com', 80, '100.10.10.0/24', 'self'),
    TrackedObject('8.8.8.8', 53, '200.20.20.0/24', 'self'),
]


while True:
    for host in tracked_objects:
        if is_alive(host.address, host.port):
            stdout.write('announce route {} next-hop {}\n'.format(host.prefix, host.nexthop))
            stdout.flush()
        else:
            stdout.write('withdraw route {} next-hop {}\n'.format(host.prefix, host.nexthop))
            stdout.flush()
    sleep(10)
```

We can create a list of tracked objects to iterate through every 10 seconds or so (depending on timeouts and availability of the services we're checking). Just picture the flexibility possible if we were to read from a database for each tracked object and do some real threading to get the timing of the checks more precise.

The next post in this series will expand on the automation of ExaBGP using python by providing an API to control routes from an external system.