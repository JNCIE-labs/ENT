### Introduction ###

The following features are covered in this lab:

 - Dynamic ARP Inspection
 - MAC Address Mobility Limits
 - MAC Address Limits
 - DHCP Snooping
 - Static Allowed MAC Addresses

Technically, some of these features are VLAN-based (DHCP Snooping, DAI), 
but they still fall into general port and switch security, so they're 
included here.

The above features provide additional security after you've implemented 
802.1x (or a measure of security before you've deployed your dot1x 
solution!).  802.1x is covered in another lab.  Firewall filters provide 
additional security but will be covered in a different lab.

### Topology ###

For today's topic, the topology is simple: SW1 connected to a hub, a host, 
and a DHCP server.  Three hosts connected to the hub (this will allow testing 
of different thresholds).

### Objectives ###

 - Configure a DHCP server
 - Configure the switch to perform DHCP Snooping
   - The trunk to the hub should be untrusted
   - The port to the DHCP server should be trusted
   - The snooping database should be persistent on reboot
 - Configure all four hosts to use the DHCP server
   - Ensure they pull IP addresses
   - Examine the snooping database in the switch
   - Reboot the switch
   - Ensure the bindings still exist
 - Modify the host connected directly to SW1 to also run a DHCP server
   - Shutdown the port facing your trusted DHCP server
   - Release the DHCP leases on the hosts connected to the hub
   - Try to renew the leases
   - Record the results
 - Place the DHCP server in a different VLAN than the hosts
   - Configure the switch as a DHCP relay agent
   - Repeat steps 1-4 under this new configuration
   - Place all devices in a single VLAN once again
 - Configure Dynamic ARP Inspect on SW1
 - Configure one of your hosts with a static IP address
   - Verify that it cannot reach other hosts in the network
 - Verify your DHCP-addressed hosts can reach other hosts in the network
 - Resolve the issue with the static IP host by adding a manual IP entry to 
the snooping database
 - Configure a MAC address mobility limit (mac-move-limit)
   - Set the limit such that moving between ports more than twice will cause 
the port to be shutdown
   - Test this by moving the host connected directly to SW1 from one port to 
another multiple times
   - Then, verify the port has been shutdown
 - On the port connected to the hub, define a static MAC address entry for 
one of the hosts
   - Verify this host can reach the DHCP server but other hosts cannot
 - Remove the static MAC address limit
 - Configure MAC address count limits
   - This limit should be applied to the port facing the hub and drop 
packets exceeding the limit but not shut the port down

