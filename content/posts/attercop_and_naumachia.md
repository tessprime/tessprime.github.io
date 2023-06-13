---
title: "Attercop_and_naumachia"
date: 2023-05-23T15:08:23-07:00
---
> Old fat spider spinning in a tree!
> Old fat spider can’t see me!
>          Attercop! Attercop!
>               Won't you stop,
> Stop your spinning and look for me?

https://www.ettercap-project.org/

So many years ago TSR decided to introduce a monster
known as 'Ettercap', which is largely believed to
share a root with Attercop.

The ettercap tool (which is a Ethernet Capture tool) has
an eastegg in its man page remembering this connection:

> The Lord Of The (Token)Ring
>       (the fellowship of the packet)
>
>       "One Ring to link them all, One Ring to ping them,
>        one Ring to bring them all and in the darkness sniff them." 

Anyhow, been poking around a bit with this due to the fact that I have
discovered that the inimitable Victor Graf has constructed a continuously
running Naumachia CTF based on his tool Naumachia.

https://naumachiactf.com/

https://github.com/nategraf/Naumachia

In this setup, Victor has made it very easy to create local pwnable networks
where much more of the network is up for grabs. Of note, ARP poisoning is
a thing you can do in this setup.

## What's ARP poisoning? What's an Ether Header
Networks send more types of packets than just ethernet packets. Packet
structure is typically a set of nested set of addreses, and the outermost
address is typically going to be the ether header, which specifies
a source MAC address and a destination MAC address. Unlike the IP component
of the packet, this will get changed when the packet leaves the local
network.

Suppose I want to ping google.com. I need to send an IP packet to 142.251.33.78.
I don't know how to get there, but I know my IP gateway does. So I'll send a
packet destined to 142.251.33.78 to my IP gateway. But to a send a packet
to that, I need to know its MAC address because it needs to be routed on
the local network.

To figure *that* out, I send an ARP packet to the broadcast address of the lan.
Every machine on the network who speaks IP (which hopefully includes the gateway,
or something has gone haywire) will receive the packet (because all devices
on the LAN speak Ether) and be able to respond if they are the owner of that IP.

Once the gateway responds, we know the MAC address of the gateway and we can
send it a nicely packaged IP packet destined for google.

However, what if the gateway dies and we swap out the old computer with a new one?
Well, we should probably tell everyone where to find the new gateway. So the
gateway can tell everyone its new mac address.

Unfortunately, the issue is this is completely unauthenticated. There's no way
for a recipient of this message to validate it came from the gateway. Hence,
ARP poisoning.

In short, a malicious node in the network (mac address 13:37:13:37)
can tell any other node (say 192.168.1.2) in the network that the MAC
address associated with 192.168.1.1 is actually 13:37:13:37, and can
even tell the router "Hello, 192.168.1.1 is now 13:37:13:37".

So yay! MITM baked into the standard. :)
