---
id: 606
title: Processing and Storing BGP Messages in MongoDB
date: 2015-06-27T18:21:05-07:00
author: Mat
layout: post
guid: https://thepacketgeek.com/?p=606
permalink: /processing-and-storing-bgp-messages-in-mongodb/
categories:
  - Coding
  - Networking
series:
  - Influence Routing Decisions with python and ExaBGP
---
<div class="seriesmeta">
  This entry is part 6 of 6 in the series <a href="https://thepacketgeek.com/series/influence-routing-decisions-with-python-and-exabgp/" class="series-26" title="Influence Routing Decisions with python and ExaBGP">Influence Routing Decisions with python and ExaBGP</a>
</div>

Now that you know how to advertise prefixes to BGP peers with ExaBGP and are familiar with how to use this to influence traffic in your network, let&#8217;s change gears and look at processing BGP messages between ExaBGP and its peers. Since ExaBGP uses JSON for message data, I figured it would be a good opportunity to use <a href="https://www.mongodb.org/" target="_blank">MongoDB</a> so the message information can easily be stored into a database for data collection and analysis.

<img class="aligncenter size-large wp-image-634" src="https://thepacketgeek.com/wp-content/uploads/2015/06/2015-06-26-15_29_46-Hangouts-mat.wood@rightside.co_-1024x479.png" alt="ExaBGP & MongoDB" width="650" height="304" srcset="https://thepacketgeek.com/wp-content/uploads/2015/06/2015-06-26-15_29_46-Hangouts-mat.wood@rightside.co_-1024x479.png 1024w, https://thepacketgeek.com/wp-content/uploads/2015/06/2015-06-26-15_29_46-Hangouts-mat.wood@rightside.co_-300x140.png 300w, https://thepacketgeek.com/wp-content/uploads/2015/06/2015-06-26-15_29_46-Hangouts-mat.wood@rightside.co_.png 1314w" sizes="(max-width: 650px) 100vw, 650px" /> 

<!--more-->In the ExaBGP configuration file you can specify the types of messages you want to receive on STDOUT and also the encoding of the messages (text or JSON). Here&#8217;s an example of configuring ExaBGP to send all BGP messages to the script running in the process section:

<pre class="lang:ini decode:true" title="conf.ini">group test { 
    router-id 172.16.2.1; 
    local-as 65000; 
    local-address 172.16.2.1; 

    process syslog { 
        run /usr/bin/python path/to/syslog.py; 
        encoder json;
        receive {
            parsed;
            update;
            neighbor-changes;
        }
    } 

    neighbor 172.16.2.10 { 
        peer-as 65000; 
    }
}</pre>

This should look real similar to previous ExaBGP config files, but note the new \`receive\` section and \`encoder\` directive under \`process syslog\`. This tells ExaBGP to output neighbor changes, updates, and all other parsed BGP messages in JSON format to STDOUT. There is also an option to output messages that ExaBGP sends to peers which you can read about <a href="https://github.com/Exa-Networks/exabgp/wiki/Configuration-:-Process" target="_blank">here</a>, but is outside the scope of this post.

Now that ExaBGP is outputting these messages we&#8217;ll look at how to use the python script specified in the \`process syslog\` section to parse the data. Here&#8217;s an example of the messages in JSON format we receive when ExaBGP peers with a router, receives a prefix, and then the peering is shutdown:

<pre class="lang:js decode:true ">// ExaBGP connects with peer 172.16.2.10
{
  "exabgp": "3.4.8",
  "time": 1435450105,
  "host": "packetgeek.local",
  "pid": "93361",
  "ppid": "92918",
  "counter": 14,
  "type": "state",
  "neighbor": {
    "ip": "172.16.2.10",
    "address": {
      "local": "172.16.2.1",
      "peer": "172.16.2.10"
    },
    "asn": {
      "local": "65000",
      "peer": "65000"
    },
    "state": "up"
  }
}

// UPDATE message sent from peer with attributes and accompanying prefix(es)
{
  "exabgp": "3.4.8",
  "time": 1435450105,
  "host": "packetgeek.local",
  "pid": "93361",
  "ppid": "92918",
  "counter": 15,
  "type": "update",
  "neighbor": {
    "ip": "172.16.2.10",
    "address": {
      "local": "172.16.2.1",
      "peer": "172.16.2.10"
    },
    "asn": {
      "local": "65000",
      "peer": "65000"
    },
    "message": {
      "update": {
        "attribute": {
          "origin": "igp",
          "med": 0,
          "local-preference": 100
        },
        "announce": {
          "ipv4 unicast": {
            "172.16.2.10": {
              "1.1.1.0/24": {},
              "10.10.0.0/24": {},
              "100.10.10.0/24": {}
            }
          }
        }
      }
    }
  }
}

// Peer sends EoR (End of RIB) to notify ExaBGP has received all prefixes
{
  "exabgp": "3.4.8",
  "time": 1435450105,
  "host": "packetgeek.local",
  "pid": "93361",
  "ppid": "92918",
  "counter": 16,
  "type": "update",
  "neighbor": {
    "ip": "172.16.2.10",
    "address": {
      "local": "172.16.2.1",
      "peer": "172.16.2.10"
    },
    "asn": {
      "local": "65000",
      "peer": "65000"
    },
    "message": {
      "eor": {
        "afi": "ipv4",
        "safi": "unicast"
      }
    }
  }
}

// Peer withdraws the 100.10.10.0/24 prefix
{
  "exabgp": "3.4.8",
  "time": 1435450170,
  "host": "packetgeek.local",
  "pid": "93361",
  "ppid": "92918",
  "counter": 17,
  "type": "update",
  "neighbor": {
    "ip": "172.16.2.10",
    "address": {
      "local": "172.16.2.1",
      "peer": "172.16.2.10"
    },
    "asn": {
      "local": "65000",
      "peer": "65000"
    },
    "message": {
      "update": {
        "withdraw": {
          "ipv4 unicast": {
            "100.10.10.0/24": {}
          }
        }
      }
    }
  }
}

// Connection with peer 172.16.2.10 is torn down
{
  "exabgp": "3.4.8",
  "time": 1435450190,
  "host": "packetgeek.local",
  "pid": "93361",
  "ppid": "92918",
  "counter": 18,
  "type": "state",
  "neighbor": {
    "ip": "172.16.2.10",
    "address": {
      "local": "172.16.2.1",
      "peer": "172.16.2.10"
    },
    "asn": {
      "local": "65000",
      "peer": "65000"
    },
    "state": "down",
    "reason": "out loop, peer reset, message [closing connection] error[the TCP connection was closed by the remote end]"
  }
}</pre>

As you can see, there&#8217;s plenty of useful information provided in these messages in a very easy to consume JSON format. Here&#8217;s a summary of the values in messages:

  * Details about ExaBGP are contained in each message (Version, localhost, process ID)
  * BGP Message type: State (related to OPEN), Notification, Update, and Keepalive
  * ABGP neighbor section to show peer  associated with the message and the message content 
      * This section will contain the route announcement/withdraw/EoR info

Just to show the process of storing these messages in our MongoDB database, let&#8217;s pretend we&#8217;re working on an app that will monitor the status of ExaBGP peers and send alerts if a peer connection is shutdown or keepalives haven&#8217;t been received in a certain amount of time. For this app, we will only need to hold onto the state and keepalive messages. Go ahead and check out <a href="https://github.com/Exa-Networks/exabgp/blob/master/etc/exabgp/processes/syslog-1.py" target="_blank">this</a> example syslog script in the ExaBGP repo that reads from STDIN, as I will be using it as a base for this next example.

Instead of storing the entire JSON message in MongoDB, I will create a summarized version with just the info needed for the app (type, peer, time, and state info). This example also converts the timestamp to a python datetime object so it&#8217;s a little easier to work with in our app. Here&#8217;s the python example to do this using the \`syslog-1.py\` script mentioned earlier as a base (I&#8217;ve preserved the original comments to help):

<pre class="lang:python decode:true" title="logtodb.py">#!/usr/bin/env python
import json
import os
from sys import stdin, stdout
from pymongo import MongoClient

###  DB Setup ###
client = MongoClient()
db = client.exabgp_db
updates = db.bgp_updates

def message_parser(line):
    # Parse JSON string  to dictionary
    temp_message = json.loads(line)

    # Convert Unix timestamp to python datetime
    timestamp = datetime.fromtimestamp(temp_message['time'])

    if temp_message['type'] == 'state':
        message = {
            'type': 'state',
            'time': timestamp,
            'peer': temp_message['neighbor']['ip'],
            'state': temp_message['neighbor']['state'],
        }

        return message

    if temp_message['type'] == 'keepalive':
        message = {
            'type': 'keepalive',
            'time': timestamp,
            'peer': temp_message['neighbor']['ip'],
        }

        return message

    # If message is a different type, ignore
    return None

counter = 0
while True:
    try:
        line = stdin.readline().strip()
        
        # When the parent dies we are seeing continual newlines, so we only access so many before stopping
        if line == "":
            counter += 1
            if counter &gt; 100:
                break
            continue
        counter = 0
        
        # Parse message, and if it's the correct type, store in the database
        message = message_parser(line)
        if message:
            updates.insert_one(message)

    except KeyboardInterrupt:
        pass
    except IOError:
        # most likely a signal during readline
        pass</pre>

* This example assumes you have <a href="http://docs.mongodb.org/manual/installation/" target="_blank">MongoDB</a> and <a href="http://api.mongodb.org/python/current/installation.html" target="_blank">pymongo</a> installed and are running it on the same host as ExaBGP with the default port.

So now if we were to connect to this database from the frontend app that will monitor BGP peer status, we can use the pymongo library to check for keepalives in the last 5 minutes from peer 172.16.2.10 like this:

<pre class="lang:python decode:true " title="check_peer_status.py">#!/usr/bin/env python
from datetime import datetime, timedelta
from pymongo import MongoClient, DESCENDING

###  DB Setup ###
client = MongoClient()
db = client.exabgp_db
updates = db.bgp_updates

def peer_is_up(peer_ip):

    # Find keepalives in last 5 minutes, True or False based on count
    has_keepalives = bool(updates.find({'peer': peer_ip, 'time': {'$gt': datetime.now() - timedelta(minutes=5)}}).count())

    # Check latest state message for this peer
    state_message = list(updates.find({'type': 'state', 'peer': peer_ip}).sort('time', DESCENDING).limit(1))
    state = state_message[0]['state']

    if has_keepalives and state == 'up':
        return True
    else:
        return False

peer = '172.16.2.10'
peer_status = peer_is_up(peer)

if peer_status:
    print 'Peer %s is up.' % peer
else:
    print 'Peer %s is down.' % peer
</pre>

This was just an introduction to working with the JSON messages from ExaBGP and storing them in a MongoDB for querying purposes. There&#8217;s plenty of potential with access to the UPDATE messages from neighbors and I&#8217;m sure a future post will cover that in more depth. Thanks for reading!