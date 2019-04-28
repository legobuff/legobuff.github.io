---
layout: post
title: happier network with pi-hole and raspberry pi
tags: misl pi-hole raspberry-pi
---
My home network started off 15 years ago with a DSL connection, WiFi access point, and two computers. Since then, the kids have grown, the devices have multiplied, one WAN connection has become two, and the home routinely burns through 1.5 terabytes of bandwidth per month.

After reviewing my network logs, I was surprised at how much network traffic was tracking/advertising related. Which brings me to [Pi-hole](https://pi-hole.net). Pi-hole advertises itself as “a black hole for internet advertisements,” which it is, but I have found it to be more than that. Essentially, it is a DNS server that will run on a [Raspberry Pi](https://www.raspberrypi.org) in your network which blacklists undesirable domains. When __any__ client on your network makes a request to an undesired domain, the domain will not resolve. This stops undesired traffic before it makes it into your network and lowers bandwidth consumption. 

To get Pi-hole up and running, I purchased a [CanaKit Raspberry Pi 3 B+ with case and power supply](https://www.amazon.com/gp/product/B07BC7BMHY/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=B07BC7BMHY&linkCode=as2&tag=legobuff-20&linkId=3cd8ef93ecbcc245ca47423215219d5d) from Amazon, used a MicroSD card I had laying around, and followed the [Get Started with Raspberry Pi](https://projects.raspberrypi.org/en/pathways/getting-started-with-raspberry-pi) and Pi-hole [Alternate Install Method 2](https://github.com/pi-hole/pi-hole/#one-step-automated-install). I then updated the default blacklists, adding many from [The Big Blocklist Collection](https://firebog.net) and [Block List Project](https://tspprs.com).

Being a heavy console gaming family, my boys did discover a few sites that needed to be whitelisted for the Xbox to work appropriately. Thankfully the Pi-hole community’s [Commonly Whitelisted Domains](https://discourse.pi-hole.net/t/commonly-whitelisted-domains/212) solved our issues.

After all that, on average I am seeing more than a __30%__ reduction in traffic. This is a __drastic__ improvement and my network is happier for it.

![](/assets/post-images/pi-hole-snapshot.png)