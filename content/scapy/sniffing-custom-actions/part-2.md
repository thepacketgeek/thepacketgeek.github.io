+++
title = "Scapy Sniffing with Custom Actions"
description = "Part 2"
#date = 2013-10-07
date = 2019-05-11
author = "Mat"
in_search_index = true
aliases = ["/scapy-sniffing-with-custom-actions-part-2/"]

[taxonomies]
tags = ["scapy", "python"]
+++

In the previous article I demonstrated how to add a custom function to change the formatting of the packet output in the console or do some sort of custom action with each packet. That example passed the function (a Callable) without any additional args. When the prn passed function is called (with each packet), it receives a single argument of the packet that was just sniffed.

Using nested functions to harness the power of closure, you can bind any number of arguments to the function that is executed on each packet by Scapy. In order to bind additional arguments to the prn function, we have to use nested functions (similar to a decorator). Check out this example, created to upload the scapy packet info to an API via the Python Requests module:

<!-- more -->

```python
#! /usr/bin/env python3

import json
import requests
from collections import Counter
from scapy.all import sniff

# define API options
url = "http://hosted.app/api/packets"
token = "supersecretusertoken"

# create parent function with passed in arguments
def custom_action(url: str, token: str):

  # uploadPacket function has access to the url & token parameters
  # because they are 'closed' in the nested function
  def upload_packet(packet):
    # upload packet, using passed arguments
    headers = {'content-type': 'application/json'}
    data = {
        'packet': packet.summary(),
        'token': token,
    }
    r = requests.post(url, data=data, headers=headers)

  return upload_packet

sniff(prn=custom_action(url, token))
```

This may seem a little strange, but here's an order-of-events explanation for what's happening:

  1. We define our `url` & `token` variables, just like in the first example.
  2. We define the `custom_action` function. This will be run when the scapy `sniff` function first runs to get the value info for the `prn` argument. Note the two parameters that we pass into `custom_action`.
  3. Inside `custom_action`, we create another function that takes the scapy implicitly passed packet as a parameter. This is the function that will upload the packet info to our API.
  4. The `upload_packet` function is nested in `custom_action` so it has access to the `url` & `token` variables because it is inside the parent function's scope.
  5. The return value of custom_action is the `upload_packet` function, so this function will be run along with every sniffed packet based on the `prn` argument. Even though the `custom_action` function is not executed to take in the `url` & `token` parameters, they are locked into the nested `upload_packet` function due to Python's capacity for ['closure'.](http://ynniv.com/blog/2007/08/closures-in-python.html "Closures in Python")
  6. After we define the `custom_action` and nested `upload_packet` functions, we run the Scapy `sniff` function with the returned value of `custom_action` passed via the `prn` argument.

Using closures to 'lock-in' any number of arguments to the `custom_action` function gives us much more flexibility. I was able to modularize my `custom_action.py` file in order to clean up my Scapy sniffing script.

Here's another example that achieves the same affect using `functools.partial` to close in the url & token arguments:

```python
#! /usr/bin/env python3

import json
import requests
from collections import Counter
from functools import partial
from scapy.all import sniff

# define API options
url = "http://hosted.app/api/packets"
token = "supersecretusertoken"


def upload_packet(api_endpoint: str, api_token: str, packet):
    # upload packet, using passed arguments
    headers = {'content-type': 'application/json'}
    data = {
        'packet': packet.summary(),
        'token': api_token,
    }
    r = requests.post(api_endpoint, data=data, headers=headers)

sniff(prn=partial(upload_packet, url, token))

```

See how I was able to move a big chunk of verbose packet protocol checking into a separate module by looking at my two python files in this project: <a title="Scapi-to-API" href="https://github.com/thepacketgeek/scapy-to-api" target="_blank" rel="noopener">Scapy-to-API</a>