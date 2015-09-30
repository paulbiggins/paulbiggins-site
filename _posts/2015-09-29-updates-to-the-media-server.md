---
layout: post
title: Updates to the Media Server
date: 2015-09-29T21:35:46-07:00
modified:
categories: 
excerpt: "Some small updates to the server to make life easier."
tags: [media, arch linux, server, blog]
image:
  feature: 2015-09-29_feature.jpg
---

I've had my media server running for a few months now, and have made a few tweaks to make life easier. I'm also burning through storage a lot faster than I anticipated, so I'll have to think about adding more hard drives soon. The first big change is moving over to [SickRage](https://github.com/SiCKRAGETV/SickRage) from Sick Beard. It's pretty straight forward, and I chose to rebuild my library rather than importing it. My favorite feature is the support for failed downloads, which Sick Beard seemed to consistently mess up. Instead of grabbing the same download that already failed, SickRage will try another one of similar quality. One thing that isn't handled very well in SickRage though is verification of downloads with parity files if you're using it in conjunction with SABnzbd. To force the correct postprocessing procedure with SABnzbd, we'll have to go and modify the url that SickRage passes to SABnzbd (per the [SABnzbd API](http://wiki.sabnzbd.org/api#toc28)).

``` sh
# /opt/sickrage/sickbeard/sab.py

...
    if category != None:
        params['cat'] = category
    params['pp'] = 1 # 1=Repair, 2=Unpack, 3=Delete
...

```

Notice that we only want to 'Repair' post processing option, so we can let SickRage unpack and delete like it wants to. This is the best fix I could come up with until SickRage decides to do something about this. Don't forget to restart the sickrage service.

Next I've created some handy little timers that take care of encoding video files into something more friendly for Kodi. I noticed that if hardware acclerated decoding is enabled, MPEG4 video playback is choppy. I want to have everything in h.264, and finding MPEG4 files is pretty easy since most of them are contained in '.avi' files. Since I already had a substantial video library, I decided to run the script in a separate screen instance so it'd be more managable. Here's what the service script looks like.

``` sh
# /etc/systemd/system/tvEncoder.service

[Service]
Type=oneshot
ExecStart=/bin/bash -c "cd /mnt/data/TV/; OLD_IFS=${IFS}; IFS=$'\n'; for file in $(find . -name *.avi); do FILE_NAME="${file%.*}"; ffmpeg -i $FILE_NAME.avi -q:v 0 -vcodec libx264 -acodec copy $FILE_NAME.mkv; rm $FILE_NAME.avi; done; IFS=${OLD_IFS}"
```

Make a similar one to process movies. We also want an associated timer file to execute this every once in a while. I thought twice per month is sufficienct since most things are encoded in h.264 these days anyway.

``` sh
# /etc/systemd/system/tvEncoder.timer

[Unit]
Description=Run TV encoder every 16ish days

[Timer]
OnCalendar=*-*-1,16 12:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

Don't forget to start and enable the timer file. That's it for now. I've been playing with some Raspberry Pi things. An update coming soon.
