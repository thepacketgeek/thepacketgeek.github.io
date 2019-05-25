---
title: Graphing packet details with PyShark, pygal, and Flask
date: 2015-04-21T08:00:12-07:00
author: Mat
layout: post
categories:
  - PyShark
---
So you&#8217;ve used PyShark to get packet statistics out of your trace files but you want to represent them in a more friendly way than just text output? &nbsp;How about using Flask and pygal to get those statistics in a graph or chart for use in a web app!

<img class="aligncenter size-large" src="{{ site.url }}/static/img/_posts/pyshark-graphing.png" alt="PyGal Example" width="650" height="367" sizes="(max-width: 650px) 100vw, 650px" /> This blog post will be a brief overview of getting some packet data in a chart and on a webpage. Please note that this will be a very simple example and not a recommended production deployment of Flask. I highly recommend using templates, flask-bootstrap, and more advanced Flask topics which you can learn about <a title="Flask" href="http://flask.pocoo.org" target="_blank">here</a>.

<!--more-->

First, let&#8217;s go over some very basic Flask app setup. We&#8217;re going to setup a single file flask app called \`app.py\` with our necessary&nbsp;imports and a single route that has a \`graph\_pcap()\` function. This route will return a basic web page with a form for users to upload a pcap file. When the file is uploaded (through an HTTP POST), our \`graph\_pcap()\` function will use PyShark to extract the packet sizes and return a chart to the user&#8217;s browser. &nbsp;The comments I&#8217;ve added below outline the logic that we fill in throughout this article. You can see the modules we will use in this project in the imports section at the top. In order to run this example, use pip in your environment (or virtualenv of course) to install the necessary modules. &nbsp;Also, for PyShark to work correctly, you&#8217;ll need to install a version of <a title="Wireshark" href="https://www.wireshark.org/download.html" target="_blank">Wireshark </a>that includes Tshark (1.10, 1.12, 1.99).

<pre class="theme:dark-terminal lang:sh decode:true ">$ pip install pygal, pyshark, flask</pre>

And here&#8217;s our app outline:

<pre class="lang:python decode:true" title="app.py">import os
import pyshark
from datetime import datetime
from flask import Flask, request, redirect, url_for
from pygal import XY
from pygal.style import LightGreenStyle
from werkzeug import secure_filename

app = Flask(__name__)

@app.route('/', methods=['GET', 'POST'])
def graph_pcap():

    if request.method == 'POST': 

        # Save uploaded file   
        
        # Process pcap, creating pygal chart with packet sizes

        # Create pygal chart instance

        # Add points to chart, render chart, and return html

    elif request.method == 'GET:
        
        # Return pcap upload form

if __name__ == '__main__':
    app.run(debug=True)</pre>

We can use the flask \`request\` object to get the uploaded file and save it to disk, where PyShark will open the pcap for analysis. For help with Flask uploads that will explain this, check out the <a title="Flask - Uploads" href="http://flask.pocoo.org/docs/0.10/patterns/fileuploads/" target="_blank">Flask doc article</a>:

<pre class="lang:default decode:true "># Save uploaded file
        traceFile = request.files['file']
        filename = secure_filename('{}{}'.format(datetime.now(), traceFile.filename))
        traceFile.save(filename)</pre>

I&#8217;m prepending the current datetime to the filename to make sure that we don&#8217;t overwrite other files as they&#8217;re uploaded and to help avoid naming conflicts. Next, we&#8217;ll create and&nbsp;populate a chart axis using a list of tuples in the (point-y, point-x) format. We&#8217;ll read in the uploaded pcap with PyShark and then iterate over the packets and store the points to the newly created list:

<pre class="lang:default decode:true"># Process pcap, creating pygal chart with packet sizes
        pkt_sizes = []
        cap = pyshark.FileCapture(filename, only_summaries=True)
        for packet in cap:
            # Create a plot point where (x=time, y=bytes)
            pkt_sizes.append((float(packet.time), int(packet.length)))</pre>

We need to create a list to store the plot points in and will use PyShark to iterate through the packets to store the packet time (absolute time in seconds from capture start) and the packet length (bytes). If you have any questions about the PyShark parts of this code I highly recommend my PyShark series starting with,&nbsp;<a title="PyShark – FileCapture and LiveCapture modules" href="http://thepacketgeek.com/pyshark-filecapture-and-livecapture-modules/" target="_blank">PyShark – FileCapture and LiveCapture modules</a>. Our next step is to create a pygal chart instance and add this packet size axis to it. <a title="pygal" href="http://pygal.org" target="_blank">Pygal</a> has some great documentation on their website, and I also highly recommend <a title="Python Library - pygal with Flask" href="http://www.blog.pythonlibrary.org/2015/04/16/using-pygal-graphs-in-flask/" target="_blank">this quick tutorial</a> on using pygal in Flask.

<pre class="lang:default decode:true "># Create pygal instance
        pkt_size_chart = XY(width=400, height=300, style=LightGreenStyle, explicit_size=True)
        pkt_size_chart.title = 'Packet Sizes'
        
        # Add points to chart and render chart html
        pkt_size_chart.add('Size', pkt_sizes)
        chart = pkt_size_chart.render()</pre>

This&nbsp;creates an&nbsp;\`XY()\` style chart object&nbsp;using one of the prebuilt styles of pygal, and also sets the size of the chart. The \`.title\` attribute will be used when rendering the chart to give the chart a title and the \`.add()\` method will add an axis to the chart. The first argument of \`.add()\` is the name of the axis which will be used in the chart legend, so graciously provided by pygal. The last step is to add the HTML template to display the chart, and also the HTML for the form when the user first browses to our page (with an HTTP GET request). Here&#8217;s the final version of this basic script for generating a chart of packet sizes of a pcap uploaded through the browser:

<pre class="lang:default decode:true " title="app.py">import os
importpyshark
fromdatetime importdatetime
from flask import Flask, request, redirect, url_for
frompygal importXY
frompygal.style importLightGreenStyle
fromwerkzeug import secure_filename

app = Flask(__name__)

#######  Flask Routes #######
@app.route('/', methods=['GET', 'POST'])
def graph_pcap():

    if request.method == 'POST': 

        # Save uploaded file
        traceFile = request.files['file']
        filename = secure_filename('{}{}'.format(datetime.now(), traceFile.filename))
        traceFile.save(filename)   

        # Processpcap, creatingpygal chart with packet sizes
        pkt_sizes = []
        pkt_window = []
        cap = pyshark.FileCapture(filename, only_summaries=True)
        for packet in cap:
            # Create a point with X=time, Y=bytes
            pkt_sizes.append((float(packet.time), int(packet.length)))

        # Create pygal instance
        pkt_size_chart = XY(width=400, height=300, style=LightGreenStyle, explicit_size=True)
        pkt_size_chart.title = 'Packet Sizes'
        
        # Add points to chart andrender html
        pkt_size_chart.add('Size', pkt_sizes)
        chart = pkt_size_chart.render()

        html = """Packet sizes over time
                
                
                    {}
                
            
            """.format(chart)

        return html

    else:
        html = """Upload PCAP
                
                
                    UploadPCAP:



<pre class="lang:default decode:true " title="app.py">                
            
            """

        return html

if __name__ == '__main__':
    app.run(debug=True)</pre>


<p>
  You can run this script from the command prompt if you go to the folder containing this `app.py` file and running the script with `python app.py`. Then you can open your browser to http://localhost:5000 and you will see the upload form:
</p>


<p>
  <img class="aligncenter size-medium" src="{{ site.url }}/static/img/_posts/pyshark-graphing-upload.png" alt="Upload Form" width="300" height="84"  sizes="(max-width: 300px) 100vw, 300px" />When we upload a pcap, our `.graph_pcap()` function will go to work and present us with this beautiful graph:
</p>


<p>
  <img class="aligncenter size-medium" src="{{ site.url }}/static/img/_posts/pyshark-chart-packet-sizes.png" alt="Packet Size Graph" width="300" height="210" sizes="(max-width: 300px) 100vw, 300px" /></a>
</p>


<p>
  As you can see, it's a fairly simple process thanks to the amazing PyShark, pygal, and Flask frameworks. I know Flask and pygal may be new to you, but there are a ton of resources out there to help you through creating your own packet analysis web app.
</p>


<p>
  Also, I'll have a new post soon that will cover adding additional axis to this chart and even more packet graphing tricks!
</p>