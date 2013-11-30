### Introduction ###

802.1x is a port-based access control mechanism that uses a RADIUS backend 
to authenticate users.  On Juniper Networks switches, this feature has a 
couple of options, the most interesting of which are probably guest VLAN 
support and the supplicant mode.  Other options include the server reject 
VLAN, which places a user in a different VLAN if he fails to authenticate; 
server fail, which tells the switch port what to do if the defined RADIUS 
server is unreachable; and static MAC, which maintains a MAC list on the 
local switch for authentication.

### Topology ###

For today's topic, the topology is simple: SW1 connected to a hub and 
RADIUS server.  Two hosts connected to the hub (this will allow testing 
of the different supplicant modes).

### Objectives ###

 - Configure a RADIUS server with two users
 - Configure the switch to use the RADIUS server for 802.1x
 - Configure VLAN in which users should be placed in the event of 
authentication failure
 - Configure the switch port facing the hub to authenticate only the first 
user
   - Try to gain access with both hosts
   - Use appropriate show commands to verify
 - Configure the switch port facing the hub to authenticate multiple users
   - Require a unique authentication success for each user
   - Try to connect with both hosts
   - Use the appropriate show commands to verify
 - Configure the port facing the hub to place all hosts into a guest VLAN 
(as if it were connected to a conference room hub)
   - Do not use 802.1x software on the hosts and attempt to connect
   - Verify with appropriate show commands.
 - Configure the switch to maintain authentication for current users in the 
event of a RADIUS server failure
   - It should not allow users who were not already authenticated to 
authenticate
   - Authenticate with one of the hosts but not the other
   - Ensure that only one host is authenticated, and then shut down the port 
facing the RADIUS server
     - Alternatively, stop the RADIUS process on the server
   - Try to connect with the user that was already authenticated
     - Verify it is still authenticated
   - Try to connect with the PC that was not previously authenticated.
   - Observe the difference.

