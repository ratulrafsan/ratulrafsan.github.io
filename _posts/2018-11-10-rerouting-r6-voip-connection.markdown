---
layout: post
title:  "Re-routing Rainbow Six: Siege VoIP connection"
date:   2018-11-10 15:00:27 +0600
categories: networking
---
On my lazy weekends, I sometimes play on-line games with my college buddies and one of my favorites is Rainbow Six:Siege. Recently my IPS changed their upstream provider and for whatever reason, it turns out that Rainbow Six's in-game VoIP server got blocked, meaning, I can't hear others and they can't hear me. Ugh!
Filed a complain about it, they said they will look into it... and that was two weeks ago, still nothing. 
So, it looks like I have to matters in my own hand and come up with a solution myself!

The obvious solution is to use VPN services. There are a lot, like A LOOOOOT of VPN services out there but most of the good ones are blocked behind pay walls. I don't blame them, its a business after all. In my hunt of a good free VPN, I stumbled across ProtonVPN (not sponsored- but I am interested *wink wink* ). They provide free VPN service with unlimited bandwidth. The only downside is that there are only a handful of free servers and the speed is pretty low. However, its enough for video games... or so I thought..

![OMFG FOOKING LAG](/assets/blog_images/fooking_lag.png)

Well now, that's a shame. One other solution I can think of is to re-route the connection to VoIP server only. So, I fired up NetLimiter and played a couple of matches. And finally narrowed the connections down to these two-

![EYYYY GOTCHU](/assets/blog_images/narrowing_the_servers.png)

Turns out Ubisoft uses Vivox for its VoIP service (Wonder what they are doing with those amazon servers...). 

Now we need a way to re-route our connection to these domains through VPN. ProtonVPN comes back to the spotlights again.. I don't know if other VPN services do this but ProtonVPN provides detailed doc on how to configure and use their VPN service without their VPN software, meaning you can use OpenVPN or Windows VPN client to connect to their network, heck they even provide OpenVPN configuration files under MIT license! I'm impressed. This makes things much easier for me.

If you have worked with OpenVPN before then you probably know how easy it is to re-route connections. Taking a quick peek at their configuration file you can see:

```
client
dev tun
proto udp

remote blah_blah_blah_remote_server.poop

remote-random
resolv-retry infinite
nobind
cipher AES-256-CBC
auth SHA512
comp-lzo no
verb 3

tun-mtu 1500
tun-mtu-extra 32
mssfix 1450
persist-key
persist-tun

reneg-sec 0

remote-cert-tls server
auth-user-pass
pull
fast-io

block-outside-dns

```
It appears they are pulling whatever data their server config provides using the ```pull``` command and then blocking all other dns services using ```block-outside-dns```. You can tell they know what they are doing. But these two lines need to go as they will interfere with my routing configuration.
Now, I can simply tell it to route all traffic though my ISP's gateway using ```route-nopull```. And specify the Vivox domains to pass through the VPN's gateway using ```route <ip address> <mask> vpn_gateway```.

Next we need the Vivox domain IPs. Sure NetLimiter gives me some IP info but most communication systems will have several IPs under its name for easy traffic management. A quick whois lookup says I was right-:

```
NetRange:       74.201.98.0 - 74.201.99.255
CIDR:           74.201.98.0/23
---------------------------------------------
NetRange:       74.201.0.0 - 74.201.255.255
CIDR:           74.201.0.0/16
```

Great, now I have the IP I need, all I need are the masks. Taking a look at https://oav.net/mirrors/cidr.html -
![cidr](/assets/blog_images/cidr.png)

mask for 23 will be ```255.255.254.0```
and mask for 16 will be ```255.255.0.0```

So, the re-routing rules will be:

```
route 74.201.98.0 255.255.254.0 vpn_gateway
route 74.201.0.0  255.255.0.0	vpn_gateway
```

And done! All theres left to do is to load this modified configuration onto OpenVPN, connect and VOILA! I CAN HEAR AND SPEAK AGAIN! 