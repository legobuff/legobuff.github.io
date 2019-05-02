---
layout: post
title: happier network with pi-hole and raspberry pi
tags: misl pi-hole raspberry-pi
---
my home network started off 15 years ago with a DSL connection, WiFi access point, and two computers. since then, the kids have grown, the devices have multiplied, one WAN connection has become two, and the home routinely burns through 1.5 terabytes of bandwidth per month.

after reviewing my network logs, I was surprised at how much network traffic was tracking/advertising related. Which brings me to [Pi-hole](https://pi-hole.net). pi-hole advertises itself as “a black hole for internet advertisements,” which it is, but I have found it to be more than that. essentially, it is a DNS server that will run on a [Raspberry Pi](https://www.raspberrypi.org) in your network which blacklists undesirable domains. when __any__ client on your network makes a request to an undesired domain, the domain will not resolve. this stops undesired traffic before it makes it into your network and lowers bandwidth consumption. 

to get pi-hole up and running, i purchased a [CanaKit Raspberry Pi 3 B+ with case and power supply](https://www.amazon.com/gp/product/B07BC7BMHY/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=B07BC7BMHY&linkCode=as2&tag=legobuff-20&linkId=3cd8ef93ecbcc245ca47423215219d5d) from amazon, used a MicroSD card I had laying around, and followed the [Get Started with Raspberry Pi](https://projects.raspberrypi.org/en/pathways/getting-started-with-raspberry-pi) and pi-hole [Alternate Install Method 2](https://github.com/pi-hole/pi-hole/#one-step-automated-install). i then updated the default blacklists, adding many from [The Big Blocklist Collection](https://firebog.net) and [Block List Project](https://tspprs.com).

being a heavy console gaming family, my boys did discover a few sites that needed to be whitelisted for the Xbox to work appropriately. Thankfully the pi-hole community’s [Commonly Whitelisted Domains](https://discourse.pi-hole.net/t/commonly-whitelisted-domains/212) solved our issues.

after all that, on average I am seeing more than a __30%__ reduction in traffic. this is a __drastic__ improvement and my network is happier for it.

![]({{ "assets/post-images/pi-hole-snapshot.png" | absolute_url }})
