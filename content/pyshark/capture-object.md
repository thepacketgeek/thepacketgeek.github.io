+++
title = "Using the Capture Object"
date = 2014-11-11
author = "Mat"
weight = 90
aliases = ["/pyshark-using-the-capture-object/"]

[taxonomies]
tags = ["pyshark", "python"]
+++

Now that we know how to use the FileCapture and LiveCapture modules to capture some packets, let's see what options we have with the returned capture object (truncated list for brevity):

```python
>>> dir(cap)
Out[3]:
['apply_on_packets',
 'close',
 'current_packet',
 'display_filter',
 'encryption',
 'input_filename',
 'next',
 'next_packet']
 ```

<!-- more -->
These are the methods/attributes that I feel are actually useful, most of the other ones are used for debugging or internally for the capture process. The `display_filter, encryption, input_filename` attributes are used for displaying parameters passed into  FileCapture or LiveCapture.

The real magic here is the `apply_on_packets()` and `next()` methods. Iteration (via `for` loop) is available because of the `next()` method, and `apply_on_packets()` is another way to iterate through the packets, passing in a function to apply to each packet:

```python
>>> cap = pyshark.FileCapture('test.pcap', keep_packets=False)
>>> def print_highest_layer(pkt)
...: print pkt.highest_layer
>>> cap.apply_on_packets(print_highest_layer)
HTTP
HTTP
HTTP
HTTP
HTTP
... (truncated)
```

This can also be used for things other than printing, such as adding the packets to a list for counting or other processing. Here's a script that will append all the packets to a list and print the count:

```python
import pyshark

def get_capture_count():
    p = pyshark.FileCapture('test.cap.pcap', keep_packets=False)

    count = []
    def counter(*args):
        count.append(args[0])

    p.apply_on_packets(counter, timeout=100000)

    return len(count)

print get_capture_count()
```

Check out the [next PyShark article](packet-object "PyShark – Using the packet Object") that covers the methods and attributes of the PyShark `packet` object.