### Tunneling IPv6 ###

In this scenario, we'll get our feet wet with IPv6 tunneling.  The IGP
has been configured to make this easier for you.  IPv4 connectivity is
provided by OSPFv2, and IPv6 connectivity will be configured by you;
RIPng will provide dynamic routing for the IPv6 domain.

Your task is to tunnel IPv6 across the backbone by making changes to
only [R2][2], [R3][3], and [R5][5].  In addition, you will need to create
policies on each applicable router to advertise all statically configured
IPv6 routes.

### Restrictions ###

 - Don't modify OSPFv2 in any router
 - Use two point-to-point GRE tunnels
 - Do not define any additional IPv6 or IPv4 addresses

### Tasks ###

 - Configure RIPng
   - RIPng should only run on interfaces configured for IPv6
   - Do not run RIPng across the backbone
 - Configure routing policies
   - [R1][1], [R6][6], and [R7][7] should advertise their loopbacks
   - [R1][1] and [R2][2] should advertise the /48 subnet
   - All other routers should have policies in place to allow RIPng to
advertise the routes appropriately
 - Create tunnels to tunnel IPv6 over an IPv4 infrastructure
   - There should be two tunnels
 - Test failover scenarios
   - Shutdown one tunnel
     - Test pings from one end of the network to the other
     - Bring the tunnel back up
   - Shutdown the second tunnel
     - Test pings from one end of the network to the other
     - Bring the tunnel back up



[1]: #
[2]: #
[3]: #
[4]: #
[5]: #
[6]: #
[7]: #
