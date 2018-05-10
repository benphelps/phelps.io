---
layout: post
title:  "Livestreaming via PS4 to a local RTMP server"
date:   2014-08-15 10:00:00 -0500
categories: nginx ps4 livestreaming
---
Live stream from your PS4 to a local computer running an RTMP server by intercepting the twitch.tv stream.

**Requirements**

 * DD-WRT enabled Router (or router with iptables compatibility)
 * *nix* Envirment
 * nginx with the nginx-rtmp-module


### IPTables

To capture the outgoing RTMP stream, I'm using NAT redirection on the Twitch.tv IP range for port 1935.  **This will cause normal twitch streaming to break.**
 
```bash
iptables -t nat -A PREROUTING -d 199.9.0.0/16 -p tcp --dport 1935 -j DNAT --to-destination <local_ip>:1935
iptables -t nat -A POSTROUTING -j MASQUERADE
```

You can ssh into your DD-WRT enabled router and add these rules, any future requests will be redirected.

### nginx-rtmp-module

Here is the config for the rtmp module and the bash script I use to convert from FLV to MP4.

```nginx
rtmp {
  server {
    listen 1935;
    application app {
      live on;
      recorder storage {
        record all;
        record_path /home/user/gameplay;
        record_unique on;
        exec_record_done /home/user/gameplay/.encode.sh $path $basename;
      }
    }
  }
}
```
```bash
#!/bin/bash
/usr/local/bin/ffmpeg -i $1 -vcodec copy -acodec copy -threads 1 /Users/phelps/gameplay/${date +"%m-%d-%y %r"}.mp4
rm $1
```
