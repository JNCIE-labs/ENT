### Preface ###

This lab should answer the following questions:

 - What bits of the configuration are specifically required for each bit of 
functionality
 - On pvlan-trunk interfaces, how is the tagging done
   - Is it MASTERVLAN, COMMUNITYVLAN / MASTERVLAN, or ISOLATIONVLAN 
   - Or is it not double tagged and instead just MASTERVLAN for promiscuous 
ports, ISOLATIONVLAN for isolated & COMMUNITYVLAN for community ports
 - What happens when you don’t configure the isolation ID

### Introduction ###

One of the topics in the JNCIE-ENT syllabus which I had no underststanding 
of at all before I started studying was Private VLANs. While this isn’t a 
huge topic – it's something that I spent a bit of today ensuring I had my 
head around. While the rest of you all probably are very familiar with them,
please bear with me as I document what I’ve discovered!

For PVLANs there are two different options. Firstly there are community 
VLANs – which is essentially a group of users that should be able to talk
to all others in their "community" plus a set of servers / routers outside
their community – but not the rest of the vlan. Second is isolation 
ports – which are ports that should only be able to talk to routers/servers
and no other users.

I’ve based this blog (and my labbing today) off of the attached topology.
Note that the "traffic mirror switch" runs a QinQ tunnel between the two
switches with a port mirror sending traffic ingressing the two ports facing
the other switches to the sniffer. Config for this is as follows:

    set interfaces ge-0/0/0 description EX2
    set interfaces ge-0/0/0 unit 0 family ethernet-switching port-mode access
    set interfaces ge-0/0/1 description EX1
    set interfaces ge-0/0/1 unit 0 family ethernet-switching port-mode access
    set interfaces ge-0/0/2 description LAPTOP
    set interfaces ge-0/0/2 unit 0 family ethernet-switching port-mode access
    set ethernet-switching-options analyzer JNCIE input ingress interface ge-0/0/1.0
    set ethernet-switching-options analyzer JNCIE input ingress interface ge-0/0/0.0
    set ethernet-switching-options analyzer JNCIE output interface ge-0/0/2.0
    set vlans QinQ vlan-id 123
    set vlans QinQ interface ge-0/0/1.0
    set vlans QinQ interface ge-0/0/0.0
    set vlans QinQ dot1q-tunneling customer-vlans native
    set vlans QinQ dot1q-tunneling customer-vlans 1-4094
    set vlans QinQ dot1q-tunneling layer2-protocol-tunneling all

> All of the hosts have been emulated using an SRX. It uses one
> routing-instance for each host.

Assuming multiple switches, for either set of functionality, you are going 
to need to do the following to create the PVLAN and the pvlan-trunk interfaces.

[EX2][1]:

    set interfaces ge-0/0/23 description MIRROR-SWITCH
    set interfaces ge-0/0/23 unit 0 family ethernet-switching port-mode trunk
    set vlans PVLAN_MASTER vlan-id 100
    set vlans PVLAN_MASTER interface ge-0/0/23.0 pvlan-trunk

If you want to make a "promiscuous" port - which can transmit traffic to every
isolation port and community vlan member, it's quite simple, you just add it 
to the vlan as a non pvlan-trunk trunk port in the vlan;

[EX1][2]:

    set interfaces ge-0/0/5 description SERVER
    set interfaces ge-0/0/5 unit 0 family ethernet-switching port-mode trunk
    set vlans PVLAN_MASTER interface ge-0/0/5.0

Right - now for the clients. Let's start with the community vlans;

I wanted to make two member community vlans on this master PVLAN. I went
about this this way.

[EX2][1]:

    set interfaces ge-0/0/0 description SALES2
    set vlans SALES vlan-id 20
    set vlans SALES interface ge-0/0/0.0
    set vlans SALES primary-vlan PVLAN_MASTER

> Don't forget to put the same config on the other EX switch, adjusting for
> descriptions and port numbers.

At this point, each SALES host could ping the SERVER host and the other SALES
host.  Seems to work well.

Having done this, I was interested to know what was actually happening when I
was moving traffic between the two hosts, so I used my sniffer to have a look.
I sent an ICMP request from SALES2 to SALES1 while sniffing the traffic 
passing between the switches.

    root@VR#run ping 192.168.66.103 source 192.168.66.100 routing-instance SALES2 count 1 rapid
    PING 192.168.66.103 (192.168.66.103): 56 data bytes
    !
    --- 192.168.66.103 ping statistics ---
    1 packets transmitted, 1 packets received, 0% packet loss
    round-trip min/avg/max/stddev = 1.372/1.372/1.372/0.000 ms

Which resulted in:

    19:29:12.157049 54:e0:32:ef:1a:80 (oui Unknown) > 54:e0:32:ef:1a:83 (oui Unknown), ethertype 802.1Q (0x8100), length 102: vlan 20, p 0, ethertype IPv4, 192.168.66.100 > 192.168.66.103: ICMP echo request, id 16716, seq 0, length 64
    19:29:12.157092 54:e0:32:ef:1a:83 (oui Unknown) > 54:e0:32:ef:1a:80 (oui Unknown), ethertype 802.1Q (0x8100), length 102: vlan 20, p 0, ethertype IPv4, 192.168.66.103 > 192.168.66.100: ICMP echo reply, id 16716, seq 0, length 64

As you can see, there is no tag stacking, so you can’t use vlan20 for 
anything else on that port.

Next, I pinged (from the same host on the VR) the server.

    root@VR#run ping 192.168.66.105 source 192.168.66.100 routing-instance SALES2 count 1 rapid
    PING 192.168.66.105 (192.168.66.105): 56 data bytes
    !
    --- 192.168.66.105 ping statistics ---
    1 packets transmitted, 1 packets received, 0% packet loss
    round-trip min/avg/max/stddev = 1.643/1.643/1.643/0.000 ms

Which resulted in:

    19:30:53.330235 54:e0:32:ef:1a:80 (oui Unknown) > 54:e0:32:ef:1a:85 (oui Unknown), ethertype 802.1Q (0x8100), length 102: vlan 20, p 0, ethertype IPv4, 192.168.66.100 > 192.168.66.105: ICMP echo request, id 16720, seq 0, length 64
    19:30:53.330333 54:e0:32:ef:1a:85 (oui Unknown) > 54:e0:32:ef:1a:80 (oui Unknown), ethertype 802.1Q (0x8100), length 102: vlan 100, p 0, ethertype IPv4, 192.168.66.105 > 192.168.66.100: ICMP echo reply, id 16720, seq 0, length 64

The interesting thing to note here is that ICMP request from the host in the 
C-VLAN is transmitted over the trunk as tag 20 (the C-VLAN vlan-id), and the 
response comes back on tag 100 (the PVLAN master vlan-id).

Okay, now for isolation ports. To turn these on, just add this command to the master vlan.

    set vlans PVLAN_MASTER no-local-switching

Then configure the ports as access ports in the vlan.

[EX1][2]:

    set interfaces ge-0/0/6 description ISOLATED-EX1
    set interfaces ge-0/0/6 unit 0 family ethernet-switching port-mode access
    set vlans PVLAN_MASTER interface ge-0/0/6.0

Note, however, that if you do just this, traffic is only isolated when it 
stays on the local switch. At this point I could not ping between hosts
"Isolated A" and "Isolated B" which are on the same EX, but could ping from 
either of them to the "Isolated" host on the other EX. This makes sense as at
this point they're just having the standard tag added to them (100) so there's
no way of the switches telling each other this is "special" traffic. To 
resolve this, add an isolation VLAN ID.

[EX1][2]:

    set vlans PVLAN_MASTER isolation-id 30

At this point I could not ping between any of the isolation ports, however
could ping to the server. Exactly as desired. Now let's how the tagging 
works.

Pinging from "Isolated A" on EX2 to the server:

    root@VR#run ping 192.168.66.105 source 192.168.66.102 routing-instance ISOLATED-EX2-A count 1 rapid
    PING 192.168.66.105 (192.168.66.105): 56 data bytes
    !
    --- 192.168.66.105 ping statistics ---
    1 packets transmitted, 1 packets received, 0% packet loss
    round-trip min/avg/max/stddev = 1.695/1.695/1.695/0.000 ms

Which resulted in:

    19:40:28.924744 54:e0:32:ef:1a:82 (oui Unknown) > 54:e0:32:ef:1a:85 (oui Unknown), ethertype 802.1Q (0x8100), length 102: vlan 30, p 0, ethertype IPv4, 192.168.66.102 > 192.168.66.105: ICMP echo request, id 16730, seq 0, length 64
    19:40:28.924892 54:e0:32:ef:1a:85 (oui Unknown) > 54:e0:32:ef:1a:82 (oui Unknown), ethertype 802.1Q (0x8100), length 102: vlan 100, p 0, ethertype IPv4, 192.168.66.105 > 192.168.66.102: ICMP echo reply, id 16730, seq 0, length 64

A couple of final points to be aware of with PVLANs:

 - PVLANs cannot be used in conjunction with Voice vlans on ports
 - If you want to allow traffic to pass between the different communities 
and isolation ports, enable proxy-arp on one or more router(s) connected to 
promiscuous ports.
   - Firewall filtering can be used on the routers offering the proxy-arp to 
control traffic between communities / isolation ports.
 - You can't configure a PVLAN and a voice vlan on the same port.

[1]: #
[2]: #
[3]: #

