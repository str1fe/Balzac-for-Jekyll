---
layout: post-no-feature
title: "Arch Linux: Share wifi connection using arp proxy"
description: "Turn your old laptop or raspberry into a wifi bridge"
category: articles
tags: [arch linux, sysadmin, network]
---

I have an old laptop collecting dust on my shelf, and i have a server, which i have been meaning to move to a different location. Preferably hidden away in a cupboard. I'm lazy and don't want to stretch RJ45 cables all around my house, so i figured i could use the wifi connection from my laptop and share internet with the server.

The laptop already had a pretty fresh installation of Arch, and the server is running ubuntu.

## Bridging the wlan
So basically what i want to do is bridge the wlan interface of my laptop with the ethernet interface. That way the server connected to the ethernet port of the laptop can use the wifi connection.

Possible solutions:

1. Use NAT(can be troublesome with two way communication i.e. connecting to Printers, homebridge or other iot devices)
2. Set up a bridge by enabling 4addr(probably a good solution if your adapter/driver supports it)
3. Use an ARP proxy

I'm running a homebridge server and could not get it to work with a NAT setup with port forwarding, so i went to search for better ways.

After a failed attempt trying to set up a bridge with 4addr mode enabled, i went on to the third option(Proxy ARP) and i managed to get it working after some trial and error.

### The laptop(wifi bridge) setup

It's possible to use dhcp, but i'm using static ip's in this example.

Enable ip forwarding and arp in terminal:

{% highlight bash linenos %}
$ echo 1 > /proc/sys/net/ipv4/conf/all/proxy_arp
$ echo 1 > /proc/sys/net/ipv4/ip_forward
{% endhighlight %}

Make the settings persist on reboot by creating config files:

`/etc/sysctl.d/30-ipforward.conf`
{% highlight bash linenos %}
net.ipv4.ip_forward=1
net.ipv6.conf.default.forwarding=1
net.ipv6.conf.all.forwarding=1
{% endhighlight %}

`/etc/sysctl.d/31-proxy-arp.conf`
{% highlight bash linenos %}
net.ipv4.conf.all.proxy_arp=1
{% endhighlight %}

Now config the wifi connection:

`/etc/netctl/ssid-of-your-wifi`
{% highlight bash linenos %}
Description='My wifi config'
Interface=wlan0
Connection=wireless
Security=wpa
ESSID=ssid-of-your-wifi
IP=static
Address='10.0.0.21/24'
Gateway='10.0.0.1'
DNS=10.0.0.1
Key=your-wifi-password
{% endhighlight %}

Config the ethernet interface:

`/etc/netctl/shared-ethernet`
{% highlight bash linenos %}
Description='Shared ethernet con'
Interface=eth0
Connection=ethernet
IP=static
Address='10.0.0.21/32'
IPCustom=('route add 10.0.0.11/32 dev eth0')  # <- The ip of the inside client
{% endhighlight %}

Now we have to set up DNS relaying

Install avahi:
{% highlight bash linenos %}
$ pacman -S avahi
{% endhighlight %}

Edit avahi config:

`/etc/avahi/avahi-daemon.conf`
{% highlight bash linenos %}
...
[reflector]
enable-reflector=yes
...
{% endhighlight %}

Now run some commands to make services start automatically on boot:

{% highlight bash linenos %}
$ netctl enable ssid-of-your-wifi
$ netctl enable shared-ethernet
$ systemctl enable avahi-daemon
{% endhighlight %}

### The inside client(aka server) setup

Assign a static ip on the ubuntu server:

`/etc/network/interfaces`
{% highlight bash linenos %}
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
# iface enp6s0 inet dhcp
auto eth0
#iface enp6s0 inet dhcp
iface eth0 inet static
address 10.0.0.11
netmask 255.255.255.0
gateway 10.0.0.1
dns-nameservers 10.0.0.1
{% endhighlight %}

And that's it. Now my server is a member of my home network.

### Overview

Device | Interface | Ip | Description
-------|-----------|----|------------
Gateway| | 10.0.0.1 | The gateway aka my router
Bridge| wlan0 | 10.0.0.21/24 | The laptop aka wifi bridge
Bridge| eth0 | 10.0.0.21/32 | The laptop aka wifi bridge
Client| eth0 | 10.0.0.11/24 |  The server aka inside client

<br />
Output of *ip route* on bridge:
{% highlight bash linenos %}
default via 10.0.0.1 dev wlan0
10.0.0.0/24 dev wlan0 proto kernel scope link src 10.0.0.21
10.0.0.11 dev eth0 scope link
{% endhighlight %}

Output of *ip route* on client:
{% highlight bash linenos %}
default via 10.0.0.1 dev eth0 onlink
10.0.0.0/24 dev eth0 proto kernel scope link src 10.0.0.11
{% endhighlight %}


Some of the resources i used:

* [http://nullroute.eu.org/~grawity/journal-2011.html#post:20110826](http://nullroute.eu.org/~grawity/journal-2011.html#post:20110826)
* [https://wiki.debian.org/BridgeNetworkConnectionsProxyArp](https://wiki.debian.org/BridgeNetworkConnectionsProxyArp)
* [https://serverfault.com/questions/152363/bridging-wlan0-to-eth0](https://serverfault.com/questions/152363/bridging-wlan0-to-eth0)
