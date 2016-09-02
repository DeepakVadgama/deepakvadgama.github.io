---
layout: post
title: IP Address basics for developers
category: blog
comments: true
excerpt: Basic understanding of IP addresses, every developer should know.
tags: networking
---

Recently a colleague of mine incorrectly used IP address of 192.168.0.245 in Google Cloud SQL console, to enable access to cloud instance from his local machine.
The change did not work since it is local IP and not an external one. I thought of jotting down few important points that I think a developer should know about IP addresses. 

## IP Address

IP address is in the form n.n.n.n where each n can range from 0 to 255. 
Whole IP address composes of 32 bits, divided into 4 groups of 8 bits each.
8 bits can represent 256 values only thus the range of 0..255. 
 
Any network is connection of computers. For these computers to talk to each other, they need to be assigned a name. 
These temporary names assigned are called IP addresses (eg: 192.168.0.245). 

One computer/device on the network is the leader who assigns these addresses. At home, that leader is our WiFi router. 
This router itself assigns itself well-known address of 192.168.0.1 so that we can access it through our browser. 
It assigns our devices distinct IP addresses as shown in the picture below.

<figure>
  <a href="{{ site.url }}/images/blog/ip/router-ip.png"><img src="{{ site.url }}/images/blog/ip/router-ip.png"></a>
</figure>

Side note: Mac address on the other hand are identifiers assigned to the hardware device and are unique across all computers and networks. 
 
## Internal IP vs external IP

Each leader (eg: Wifi router) of a network which assigns IP address to computer, is itself just a recipient/client for leader of a larger network (eg: ISP).

For example: Your WiFi router can assign IP addresses 5 smartphone, 2 laptops, 2 tables at home. 
Though, for your ISP (Comcast, Time Warner, Hathway etc), your router is a single computer. 
Thus your ISP may think you are using Google, YouTube, Facebook all at the same time while actually each of the device might be using one site. 
This is because it is not aware of computers/phones behind your router.
 
<figure>
  <a href="{{ site.url }}/images/blog/ip/router-isp.png"><img src="{{ site.url }}/images/blog/ip/router-isp.png"></a>
</figure>

 This is precisely why external IP (your IP as seen by external network) is different.
 
<figure>
  <a href="{{ site.url }}/images/blog/ip/ipaddress-external.png"><img src="{{ site.url }}/images/blog/ip/ipaddress-external.png"></a>
</figure>
 
## Internet = network of networks
 
This recursive hierarchy can keep growing and each level has its own name, which we need not remember. 
  If you look at the whole internet, its consists of billions of nodes in a hierarchy of few core large networks connected to each other using what's called Internet Backbone.  

<figure>
 <a href="{{ site.url }}/images/blog/ip/internet.png"><img src="{{ site.url }}/images/blog/ip/internet.png"></a>
</figure>


## Dynamic vs static IP address

The IP addresses are typically dynamic. Every time you reconnect to your WiFi and check your IP address it is different. This is okay, since it is only
the leader (in our case, the router) that needs to know a computer's IP address. 

But in some cases, the computers which host websites for example, needs to be accessible by all. These cannot have dynamic IP addresses. 
On the otherhand, we cannot store and remember IP addresses of all the websites on our device like smartphone. 

The solution to this problem, is to map names of websites to the IP addresses of computers. This is called DNS - Domain Naming System.

<figure>
 <a href="{{ site.url }}/images/blog/ip/dns.png"><img src="{{ site.url }}/images/blog/ip/dns.png"></a>
</figure>

Thus our devices only need to know the IP address of DNS server (this is also configurable in your router). We can contact the DNS server, ask it to give us IP address of say, Google.com and it returns 
IP address of Google.com, which we can connect directly to.

## CIDR format - Midway of static vs dynamic

Such cases of static IP addresses are rare for personal computers like ours. Though, some ISPs do 
provide static IP address if requested. 

If you are using cloud services for development, you might need access to operate say SQL server from your local SQL client. 
  By default, for security purposes access is denied for any external computer. You need to add your IP address explicitly in cloud configuration.
  
  Since you cannot add dynamic IP (it will keep changing) and you don't have static IP you can use intermediate format called [CIDR format](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing#IPv4_CIDR_blocks).
  So if my ISP always assigns me IP address of 20.54.xxx.xxx I can use format 20.54.0.0/16. This says, allow any IP address starting with 20.54 and ignore last 16 bits.   

<figure>
 <a href="{{ site.url }}/images/blog/ip/sql-cloud.png"><img src="{{ site.url }}/images/blog/ip/sql-cloud.png"></a>
</figure>

Note: As far as possible, using complete IP addresses in such configuration since open access, or access to complete range of IP increases security risk.

## IPv6

The number of devices (eg: websites) which need static addresses have increased rapidly over the years. But the number of IPv4 addresses possible though large, are limited (2^32=4.3 billion). 

[IPv6](https://en.wikipedia.org/wiki/IPv6) was created to address this issue. IPv6 uses 128 bit addresses the capacity to 2^128, which is a massive number which we will probably never exhaust.

Then why have we not started using IPv6 everywhere? Thats because each device that participates in flow of internet (routers, proxies, servers, devices), needs
 to be able to understand IPv6. So though the transition to IPv6 has begun, it will take few more years to complete. 
  
Feel free to comment if you have any related queries. 