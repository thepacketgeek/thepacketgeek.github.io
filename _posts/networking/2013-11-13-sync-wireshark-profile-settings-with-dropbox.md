---
id: 176
title: Sync Wireshark Profile Settings with Dropbox
date: 2013-11-13T17:02:42-07:00
author: Mat
layout: post
guid: http://thepacketgeek.com/?p=176
permalink: /sync-wireshark-profile-settings-with-dropbox/
categories:
  - Networking
tags:
  - Wireshark
---
In this day and age you probably have more than one computer (laptop, VM, home desktop??). Also, if you&#8217;re like me you probably have Wireshark installed on anything you can get your hands on! It can be a bit of a pain to keep your favorite Wireshark settings such as protocol options, coloring rules, and saved display filters up to date with each Wireshark installation. Using Dropbox (or a similar service) you can easily keep your Wireshark profiles in sync on all computers.<!--more-->

**1.** Create a new folder to save your Wireshark profiles. In my Dropbox folder, I created a `Dropbox/Utilities/WiresharkSettings/` folder.

**2.** Find where Wireshark is saving your current settings. I did this on my main work computer since I know my settings are up to date there. Open Wireshark and click Help > About Wireshark. In the window that opens, click the Folders tab. We&#8217;re looking for the Personal Configuration folder. Once you make note of the folder in use make sure to close Wireshark.

<p style="text-align: center;">
  <img class="wp-image-177 aligncenter" src="//thepacketgeek.com/wp-content/uploads/2013/11/Screenshot-2013-11-13-16.23.02-240x300.png" alt="Wireshark About" width="168" height="210" srcset="https://thepacketgeek.com/wp-content/uploads/2013/11/Screenshot-2013-11-13-16.23.02-240x300.png 240w, https://thepacketgeek.com/wp-content/uploads/2013/11/Screenshot-2013-11-13-16.23.02.png 456w" sizes="(max-width: 168px) 100vw, 168px" />
</p>

<div style="clear: both;">
  <a href="http://thepacketgeek.com/wp-content/uploads/2013/11/Wireshark-Folders.png"><img class="aligncenter size-large wp-image-189" src="//thepacketgeek.com/wp-content/uploads/2013/11/Wireshark-Folders-1024x457.png" alt="Wireshark Folders" width="650" height="290" srcset="https://thepacketgeek.com/wp-content/uploads/2013/11/Wireshark-Folders-1024x457.png 1024w, https://thepacketgeek.com/wp-content/uploads/2013/11/Wireshark-Folders-300x134.png 300w, https://thepacketgeek.com/wp-content/uploads/2013/11/Wireshark-Folders.png 1300w" sizes="(max-width: 650px) 100vw, 650px" /></a>
</div>

**3.** Copy the contents of the Wireshark Personal Configuration folder to your newly created Dropbox folder.  
On Linux or OS X:

    $ cp .wireshark/* ~/Dropbox/Utilities/WiresharkSettings/

On Windows:

<pre class="lang:default decode:true">&gt; copy %APPDATA%\Wireshark\* %USERPROFILE%\Dropbox\Utilities\WiresharkSettings\</pre>

&nbsp;

**4.** To create the symlink so Wireshark looks at the Dropbox folder instead of the local settings folder. With Wireshark closed, run the following commands using the folder path of your Dropbox Wireshark settings folder (if different than mine).

On Linux or OS X:

    # Backup local settings
    $ mv .wireshark/profiles .wireshark/profiles.OLD
    $ mv .wireshark/plugins .wireshark/plugins.OLD
    # Create the Symbolic links to the Dropbox folder
    $ ln -s ~/Dropbox/Utilities/WiresharkSettings/plugins ./.wireshark/plugins
    $ ln -s ~/Dropbox/Utilities/WiresharkSettings/profiles ./.wireshark/profiles

On Windows:

<pre class="lang:default decode:true  "># Backup local settings
&gt; move %APPDATA%\Wireshark\plugins %APPDATA%\Wireshark\plugins.OLD
&gt; move %APPDATA%\Wireshark\profiles %APPDATA%\Wireshark\profiles.OLD
# Create the Symbolic links to the Dropbox folder
&gt; mklink /D %APPDATA%\Wireshark\plugins "%USERPROFILE%\Dropbox\Utilities\WiresharkSettings\plugins"
&gt; mklink /D %APPDATA%\Wireshark\profiles "%USERPROFILE%\Dropbox\Utilities\WiresharkSettings\profiles"</pre>

&nbsp;

<p class="caption">
  Note: quotation marks are only required if your folder name has a space in it.
</p>

**5.** Now, when you open Wireshark you will see that your settings and profiles are still there. Not much has changed yet, but the magic happens when you run step 4 on an additional computer.

Since we&#8217;re just linking the plugins and profiles folders in the Personal Configuration folder, this will only include the profiles and plugins, not the &#8216;recent&#8217; saved settings such as recently opened files, capture filters, display filters, viewing options, etc.. &nbsp;I find it particularly helpful to have all of those recent items synced but had issues with Windows trying to overwrite them. If you&#8217;d rather have everything sync (and deal with those issues), just make a symlink to the entire `.wireshark` or `Wireshark` folders.

This has helped me keep my coloring rules and saved filters in sync across multiple Wireshark installations, and I hope you find it just as useful!