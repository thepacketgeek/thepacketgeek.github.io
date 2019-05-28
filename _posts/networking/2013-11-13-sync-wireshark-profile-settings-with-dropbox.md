---
title: Sync Wireshark Profile Settings with Dropbox
date: 2013-11-13T17:02:42-07:00
author: Mat
layout: post
permalink: /sync-wireshark-profile-settings-with-dropbox/
categories:
  - Networking
---
In this day and age you probably have more than one computer (laptop, VM, home desktop??). Also, if you're like me you probably have Wireshark installed on anything you can get your hands on! It can be a bit of a pain to keep your favorite Wireshark settings such as protocol options, coloring rules, and saved display filters up to date with each Wireshark installation. Using Dropbox (or a similar service) you can easily keep your Wireshark profiles in sync on all computers.<!--more-->

**1.** Create a new folder to save your Wireshark profiles. In my Dropbox folder, I created a `Dropbox/Utilities/WiresharkSettings/` folder.

**2.** Find where Wireshark is saving your current settings. I did this on my main work computer since I know my settings are up to date there. Open Wireshark and click Help > About Wireshark. In the window that opens, click the Folders tab. We're looking for the Personal Configuration folder. Once you make note of the folder in use make sure to close Wireshark.

<p style="text-align: center;">
  <img class="wp-image-177 aligncenter" src="{{ site.url }}/static/img/wireshark-about.png"  alt="Wireshark About" width="168" height="210" sizes="(max-width: 168px) 100vw, 168px" />
</p>

<div style="clear: both;">
  <img class="aligncenter size-large wp-image-189" src="{{ site.url }}/static/img/wireshark-about-mac.png" alt="Wireshark Folders" width="650" height="290" sizes="(max-width: 650px) 100vw, 650px" />
</div>

**3.** Copy the contents of the Wireshark Personal Configuration folder to your newly created Dropbox folder.  
On Linux or OS X:

```
$ cp .wireshark/* ~/Dropbox/Utilities/WiresharkSettings/
```

On Windows:

```
> copy %APPDATA%\Wireshark\* %USERPROFILE%\Dropbox\Utilities\WiresharkSettings\
```


**4.** To create the symlink so Wireshark looks at the Dropbox folder instead of the local settings folder. With Wireshark closed, run the following commands using the folder path of your Dropbox Wireshark settings folder (if different than mine).

On Linux or OS X:

```bash
# Backup local settings
$ mv .wireshark/profiles .wireshark/profiles.OLD
$ mv .wireshark/plugins .wireshark/plugins.OLD
# Create the Symbolic links to the Dropbox folder
$ ln -s ~/Dropbox/Utilities/WiresharkSettings/plugins ./.wireshark/plugins
$ ln -s ~/Dropbox/Utilities/WiresharkSettings/profiles ./.wireshark/profiles
```

On Windows:

```bash
# Backup local settings
> move %APPDATA%\Wireshark\plugins %APPDATA%\Wireshark\plugins.OLD
> move %APPDATA%\Wireshark\profiles %APPDATA%\Wireshark\profiles.OLD
# Create the Symbolic links to the Dropbox folder
> mklink /D %APPDATA%\Wireshark\plugins "%USERPROFILE%\Dropbox\Utilities\WiresharkSettings\plugins"
> mklink /D %APPDATA%\Wireshark\profiles "%USERPROFILE%\Dropbox\Utilities\WiresharkSettings\profiles"
```

<p class="caption">
  Note: quotation marks are only required if your folder name has a space in it.
</p>

**5.** Now, when you open Wireshark you will see that your settings and profiles are still there. Not much has changed yet, but the magic happens when you run step 4 on an additional computer.

Since we're just linking the plugins and profiles folders in the Personal Configuration folder, this will only include the profiles and plugins, not the `recent` saved settings such as recently opened files, capture filters, display filters, viewing options, etc.. I find it particularly helpful to have all of those recent items synced but had issues with Windows trying to overwrite them. If you'd rather have everything sync (and deal with those issues), just make a symlink to the entire `.wireshark` or `Wireshark` folders.

This has helped me keep my coloring rules and saved filters in sync across multiple Wireshark installations, and I hope you find it just as useful!