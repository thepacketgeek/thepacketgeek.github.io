+++
title = "Give ExaBGP an HTTP API"
date = 2015-05-22
author = "Mat"
aliases = ["/give-exabgp-an-http-api-with-flask/"]
weight = 80

[taxonomies]
tags = ["exabgp", "python"]
+++

We've covered how to setup ExaBGP and peer with a router, and then how to use python to add  and remove advertised routes in BGP either with static definitions or dynamically through health checking. There may be some of you out there with some sort of application that is already monitoring routes and you're trying to figure out how to connect it with ExaBGP for the actual interaction part? Well, what if we add an HTTP API to ExaBGP to give programmatic access to ExaBGP from some external utility? I'll go over two ways to do this using the python built-in SimpleHTTPServer or <a href="//flask.pocoo.org" target="_blank">Flask</a>.

<!-- more -->
### Using SimpleHTTPServer

This option is great since you don't need to install any extra modules. It seems to be pretty lightweight and should be enough for some basic HTTP interaction with ExaBGP. The basic functions that we need this API to do are receive a form via HTTP POST and print that to STDOUT. Since ExaBGP is executing the python script, the STDOUT output will be visible to the ExaBGP process. After the ExaBGP command is printed, we'll return the command to the browser as confirmation that the call was successful. Here's the example:

```python
import cgi
import SimpleHTTPServer
import SocketServer
from sys import stdout

PORT = 5001

class ServerHandler(SimpleHTTPServer.SimpleHTTPRequestHandler):

    def createResponse(self, command):
        """ Send command string back as confirmation """
        self.send_response(200)
        self.send_header('Content-Type', 'application/text')
        self.end_headers()
        self.wfile.write(command)
        self.wfile.close()

    def do_POST(self):
        """ Process command from POST and output to STDOUT """
        form = cgi.FieldStorage(
            fp=self.rfile,
            headers=self.headers,
            environ={'REQUEST_METHOD':'POST'})
        command = form.getvalue('command')
        stdout.write('%s\n' % command)
        stdout.flush()
        self.createResponse('Success: %s' % command)

handler = ServerHandler
httpd = SocketServer.TCPServer(('', PORT), handler)
stdout.write('serving at port %s\n' % PORT)
stdout.flush()
httpd.serve_forever()
```

### Using Flask

If you have bigger plans for the HTTP side of things and want to work with a web framework like Flask, this example is for you. The biggest difference is that you will have to install <a href="//flask.pocoo.org" target="_blank">Flask</a> and its dependencies, although that's easy:

`$ pip install flask`

Now we'll create a Flask `http_api.py` file to listen for prefix commands via HTTP POST calls and print them to STDOUT so ExaBGP can do its magic:

```python
from flask import Flask, request
from sys import stdout

app = Flask(__name__)

# Setup a command route to listen for prefix advertisements 
@app.route('/', methods=['POST'])
def command():

	command = request.form['command']
	stdout.write('%s\n' % command)
	stdout.flush()

	return '%s\n' % command

if __name__ == '__main__':
    app.run()
```

Before you run off and put this on your production systems I have a couple disclaimers:

  * This script uses the built-in Flask debug HTTP server. It's fine for lab use, but I would use gunicorn and nginx for real heavy lifting.
  * The script doesn't do any validation of the command. A better script would make sure it's a valid command and prefix.
  * Flask defaults to listening on the localhost address on TCP port 5000. You can change this though and I highly recommend reading <a href="http://flask.pocoo.org/docs/0.10/quickstart/#quickstart" target="_blank">Flask's QuickStart article</a> to familiarize yourself with the many options.

### Hooking the HTTP API up to the ExaBGP process

Update the `conf.ini` file to run this script instead of our previous health check example:

```ini
group test {                         # An arbitrary group name
    router-id 172.16.2.1;            # Our local router-id
    
    neighbor 172.16.2.128 {          # Remote neighbor to peer with
        local-address 172.16.2.1;    # Our local update-source
        local-as 65000;              # Our local AS
        peer-as 65000;               # Peer's AS
    }
 
    process http-api {
        run \path\to\python \path\to\http_api.py;
    }
}
```

And now when we run `$ exabgp conf.ini`, you'll see the confirmation of running the python script at the end of the output, along with the debug output of both Flask or SimpleHTTPServer:

```sh
... | INFO     | 20432  | reactor       | New peer setup: neighbor 172.16.2.128 local-ip 172.16.2.1 local-as 65000 peer-as 65000 router-id 172.16.2.1 family-allowed in-open
... | WARNING  | 20432  | configuration | Loaded new configuration successfully
... | INFO     | 20432  | processes     | Forked process http-api
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
 ```

Once either HTTP service is running (somewhat wrapped by the ExaBGP process), we can make HTTP POST calls via command line (curl, wget) or via a GUI HTTP tool (Postman). I'll show a quick example of both:

Curl:

```sh
packetgeek:~ mat$ curl --form "command=announce route 100.10.0.0/24 next-hop self" http://localhost:5000/ 
announce route 100.10.0.0/24 next-hop self
```

Postman:
~[](exabgp-postman.png)

When we run `show bgp` on the router, we can see our advertised network:

```sh
CRS1#sh bgp | b Network
     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.10.0.0/24     0.0.0.0                  0         32768 i
 *>i 100.10.0.0/24    172.16.2.1                    100      0 i
```

So, there's a real quick and dirty way to allow external calls to ExaBGP for your RIB manipulation. The next post will cover more advanced peering and advertising options with ExaBGP.