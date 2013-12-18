# OSPFv3 Lab Walkthrough #

Before beginning, please load the initial configurations from the `conf/init/` folder.

## Enabling IPv6 ##

Enabling IPv6 on all routers should be done with the following commands:

    configure exclusive
    set interfaces fe-0/0/0 unit 0 family inet6
    set interfaces fe-0/0/1 unit 0 family inet6
    commit and-quit

## Configure Firewall Filter ##

To start, the firewall filter should be configured to ensure only the
required services are allowed and all others are filtered.  The configuration
is the same on all routers:

    configure exclusive
    set firewall family inet6 filter PROTECT_RE term else then discard
    set interfaces lo0.0 family inet6 filter input PROTECT_RE
    commit and-quit

## Allowing ICMPv6 ##

We're going to need ICMPv6 for our tests, so that needs to be allowed
through the filter.  On all routers:

    configure exclusive
    edit firewall family inet6 filter PROTECT_RE
    set term if-icmpv6 from next-header icmpv6
    set term if-icmpv6 then accept
    insert term else after term if-icmpv6
    commit and-quit

## Configuring OSPFv3  ## 

Configuring OSPFv3 is also simple.  We'll configure it without any
authentication, ensure it is up, and then turn authentication on
and check them again.

### Filter Change ###

We'll do the filter changes first so that we can make incremental changes to
OSPF and monitor the effects and outcome.

The change is the same in all routers:

    configure exclusive
    edit firewall family inet6 filter PROTECT_RE
    set term if-ospf from next-header ospf
    set term if-ospf from source-address fe80::/64
    set term if-ospf then accept
    insert term else after term if-ospf
    commit and-quit


### Basic Configuration ###

As there is no IPv4 configuration involved, it's worth mentioning that
you must manually define a Router ID.  The router will not select
its own as the rules for selecting a Router ID when one is not defined
apply strictly to IPv4 addresses.

The following configuration is common across all routers:

    configure exclusive
    edit protocols ospf3 area 0.0.0.0
    set interface fe-0/0/0.0 interface-type p2p
    set interface fe-0/0/1.0 interface-type p2p
    set interface lo0.0 passive
    commit and-quit

The following configuration is specific to individual routers.

`VULCAN`:

    configure exclusive
    set routing-options router-id 1.1.1.1
    commit and-quit

`ROMULAN`:

    configure exclusive
    set routing-options router-id 2.2.2.2
    commit and-quit

`BORG`:

    configure exclusive
    set routing-options router-id 3.3.3.3
    commit and-quit

`KLINGON`:

    configure exclusive
    set routing-options router-id 4.4.4.4
    commit and-quit

#### Verifying OSPFv3 ####

OSPFv3 verification is pretty straightforward.  In each of the routers,
enter the following:

    show ospf3 neighbor detail
    show route 2001:db8::/64
    ping 2001:db8::1 count 1
    ping 2001:db8::2 count 1
    ping 2001:db8::3 count 1
    ping 2001:db8::4 count 1

`Example`:

    lab@VULCAN> show ospf3 neighbor detail
    ID               Interface              State     Pri   Dead
    2.2.2.2          em0.0                  Full      128     33
      Neighbor-address fe80::2ab:b1ff:fec4:7200
      Area 0.0.0.0, opt 0x13, OSPF3-Intf-Index 2
      DR-ID 0.0.0.0, BDR-ID 0.0.0.0
      Up 00:24:31, adjacent 00:24:31
    3.3.3.3          em1.0                  Full      128     33
      Neighbor-address fe80::2ab:44ff:fe08:f801
      Area 0.0.0.0, opt 0x13, OSPF3-Intf-Index 3
      DR-ID 0.0.0.0, BDR-ID 0.0.0.0
      Up 00:24:30, adjacent 00:24:30
    
    lab@VULCAN> show route 2001:db8::/64
    
    inet6.0: 9 destinations, 10 routes (9 active, 0 holddown, 0 hidden)
    + = Active Route, - = Last Active, * = Both
    
    2001:db8::1/128    *[Direct/0] 01:40:49
                        > via lo0.0
    2001:db8::2/128    *[OSPF3/10] 00:24:31, metric 1
                        > to fe80::2ab:b1ff:fec4:7200 via em0.0
    2001:db8::3/128    *[OSPF3/10] 00:24:31, metric 1
                        > to fe80::2ab:44ff:fe08:f801 via em1.0
    2001:db8::4/128    *[OSPF3/10] 00:24:31, metric 2
                        > to fe80::2ab:b1ff:fec4:7200 via em0.0
                          to fe80::2ab:44ff:fe08:f801 via em1.0
    
    lab@VULCAN> ping 2001:db8::1 count 1
    PING6(56=40+8+8 bytes) 2001:db8::1 --> 2001:db8::1
    16 bytes from 2001:db8::1, icmp_seq=0 hlim=64 time=0.738 ms
    
    --- 2001:db8::1 ping6 statistics ---
    1 packets transmitted, 1 packets received, 0% packet loss
    round-trip min/avg/max/std-dev = 0.738/0.738/0.738/0.000 ms
    
    lab@VULCAN> ping 2001:db8::2 count 1
    PING6(56=40+8+8 bytes) 2001:db8::1 --> 2001:db8::2
    16 bytes from 2001:db8::2, icmp_seq=0 hlim=64 time=1.431 ms
    
    --- 2001:db8::2 ping6 statistics ---
    1 packets transmitted, 1 packets received, 0% packet loss
    round-trip min/avg/max/std-dev = 1.431/1.431/1.431/0.000 ms
    
    lab@VULCAN> ping 2001:db8::3 count 1
    PING6(56=40+8+8 bytes) 2001:db8::1 --> 2001:db8::3
    16 bytes from 2001:db8::3, icmp_seq=0 hlim=64 time=1.905 ms
    
    --- 2001:db8::3 ping6 statistics ---
    1 packets transmitted, 1 packets received, 0% packet loss
    round-trip min/avg/max/std-dev = 1.905/1.905/1.905/0.000 ms
    
    lab@VULCAN> ping 2001:db8::4 count 1
    PING6(56=40+8+8 bytes) 2001:db8::1 --> 2001:db8::4
    16 bytes from 2001:db8::4, icmp_seq=0 hlim=63 time=2.867 ms
    
    --- 2001:db8::4 ping6 statistics ---
    1 packets transmitted, 1 packets received, 0% packet loss
    round-trip min/avg/max/std-dev = 2.867/2.867/2.867/0.000 ms
    
    lab@VULCAN>

### Configuring OSPFv3 Authentication ###

Configuration is the same on all routers:

    configure exclusive
    edit security ipsec security-association ospf3
    set mode transport
    edit manual direction bidirectional
    set protocol ah
    set spi 256
    set authentication algorithm hmac-sha1-96
    set authentication key ascii-text "$9$sd2oGUjqfQns2aUHmF3ylKvX-2gJDHqKMGiHqzFp0B1SeLX-ds4xNjH.mTQ"
    top edit protocols ospf3 area 0.0.0.0
    set interface fe-0/0/0.0 ipsec-sa ospf3
    set interface fe-0/0/1.0 ipsec-sa ospf3
    commit and-quit

#### Verify OSPFv3 Authentication ####

On all routers:

    show ospf3 interface detail
    show ipsec security-associations detail
    show route 2001:db8::/64
    ping 2001:db8::1 count 1
    ping 2001:db8::2 count 1
    ping 2001:db8::3 count 1
    ping 2001:db8::4 count 1

`Example`:

    lab@VULCAN> show ospf3 interface detail
    Interface           State   Area            DR ID           BDR ID          Nbrs
    em0.0               PtToPt  0.0.0.0         0.0.0.0         0.0.0.0            1
      Address fe80::2ab:7dff:fee3:8200, Prefix-length 64
      OSPF3-Intf-index 2, Type P2P, MTU 1500, Cost 1
      Adj count: 1, Router LSA ID: 0
      Hello 10, Dead 40, ReXmit 5, Not Stub
      IPSec SA name: ospf3
      Protection type: None
    em1.0               PtToPt  0.0.0.0         0.0.0.0         0.0.0.0            1
      Address fe80::2ab:7dff:fee3:8201, Prefix-length 64
      OSPF3-Intf-index 3, Type P2P, MTU 1500, Cost 1
      Adj count: 1, Router LSA ID: 0
      Hello 10, Dead 40, ReXmit 5, Not Stub
      IPSec SA name: ospf3
      Protection type: None
    lo0.0               DRother 0.0.0.0         0.0.0.0         0.0.0.0            0
      Address fe80::2ab:7d0f:fce3:8200, Prefix-length 128
      OSPF3-Intf-index 1, Type LAN, MTU 65535, Cost 0
      Adj count: 0, Router LSA ID: -, Passive
      Hello 10, Dead 40, ReXmit 5, Not Stub
      Protection type: None

    lab@VULCAN> show ipsec security-associations detail
    Security association: ospf3

        Direction: inbound, SPI: 256, AUX-SPI: 0
        Mode: transport, Type: manual, State: Installed
        Protocol: AH, Authentication: hmac-sha1-96, Encryption: None
        Anti-replay service: Disabled

        Direction: outbound, SPI: 256, AUX-SPI: 0
        Mode: transport, Type: manual, State: Installed
        Protocol: AH, Authentication: hmac-sha1-96, Encryption: None
        Anti-replay service: Disabled

    lab@VULCAN> show route 2001:db8::/64 terse

    inet6.0: 9 destinations, 10 routes (9 active, 0 holddown, 0 hidden)
    + = Active Route, - = Last Active, * = Both

    A Destination        P Prf   Metric 1   Metric 2  Next hop         AS path
    * 2001:db8::1/128    D   0                       >lo0.0
    * 2001:db8::2/128    O  10          1            >fe80::2ab:b1ff:fec4:7200
    * 2001:db8::3/128    O  10          1            >fe80::2ab:44ff:fe08:f801
    * 2001:db8::4/128    O  10          2            >fe80::2ab:b1ff:fec4:7200
                                                      fe80::2ab:44ff:fe08:f801

    lab@VULCAN> ping 2001:db8::1 count 1
    PING6(56=40+8+8 bytes) 2001:db8::1 --> 2001:db8::1
    16 bytes from 2001:db8::1, icmp_seq=0 hlim=64 time=0.599 ms

    --- 2001:db8::1 ping6 statistics ---
    1 packets transmitted, 1 packets received, 0% packet loss
    round-trip min/avg/max/std-dev = 0.599/0.599/0.599/0.000 ms

    lab@VULCAN> ping 2001:db8::2 count 1
    PING6(56=40+8+8 bytes) 2001:db8::1 --> 2001:db8::2
    16 bytes from 2001:db8::2, icmp_seq=0 hlim=64 time=1.452 ms

    --- 2001:db8::2 ping6 statistics ---
    1 packets transmitted, 1 packets received, 0% packet loss
    round-trip min/avg/max/std-dev = 1.452/1.452/1.452/0.000 ms

    lab@VULCAN> ping 2001:db8::3 count 1
    PING6(56=40+8+8 bytes) 2001:db8::1 --> 2001:db8::3
    16 bytes from 2001:db8::3, icmp_seq=0 hlim=64 time=1.354 ms

    --- 2001:db8::3 ping6 statistics ---
    1 packets transmitted, 1 packets received, 0% packet loss
    round-trip min/avg/max/std-dev = 1.354/1.354/1.354/0.000 ms

    lab@VULCAN> ping 2001:db8::4 count 1
    PING6(56=40+8+8 bytes) 2001:db8::1 --> 2001:db8::4
    16 bytes from 2001:db8::4, icmp_seq=0 hlim=63 time=2.682 ms

    --- 2001:db8::4 ping6 statistics ---
    1 packets transmitted, 1 packets received, 0% packet loss
    round-trip min/avg/max/std-dev = 2.682/2.682/2.682/0.000 ms

    lab@VULCAN>

## Configuring BFD ##

Since we're going to rely on BFD to maintain OSPFv3 relationships, it makes
sense to configure it next.  This will be a multi-step process as we will
need to configure authentication and add an exception to the firewall filter.

### Filter Changes ###

We'll do the filter changes first so that we can make incremental changes to
BFD and monitor the effects and outcome.

The change is the same in all routers:

    configure exclusive
    edit firewall family inet6 filter PROTECT_RE
    set term if-bfd from next-header udp
    set term if-bfd from port 3784
    set term if-bfd then accept
    insert term else after term if-bfd
    commit and-quit

### BFD Configuration ###

As usual, the following configuration changes apply to all routers:

    configure exclusive
    edit protocols ospf3 area 0.0.0.0
    set interfaces fe-0/0/0.0 bfd-liveness-detection minimum-interval 300
    set interfaces fe-0/0/0.0 bfd-liveness-detection full-neighbors-only
    set interfaces fe-0/0/1.0 bfd-liveness-detection minimum-interval 300
    set interfaces fe-0/0/1.0 bfd-liveness-detection full-neighbors-only
    commit and-quit

#### BFD Verification ####

Verify OSPF is still up and that BFD is up now as well:

    show ospf3 neighbor detail
    show bfd session

`Example`:

    lab@VULCAN> show ospf3 neighbor detail
    ID               Interface              State     Pri   Dead
    2.2.2.2          em0.0                  Full      128     37
      Neighbor-address fe80::2ab:b1ff:fec4:7200
      Area 0.0.0.0, opt 0x13, OSPF3-Intf-Index 2
      DR-ID 0.0.0.0, BDR-ID 0.0.0.0
      Up 00:31:04, adjacent 00:31:04
    3.3.3.3          em1.0                  Full      128     38
      Neighbor-address fe80::2ab:44ff:fe08:f801
      Area 0.0.0.0, opt 0x13, OSPF3-Intf-Index 3
      DR-ID 0.0.0.0, BDR-ID 0.0.0.0
      Up 00:31:03, adjacent 00:31:03

    lab@VULCAN> show bfd session
                                                      Detect   Transmit
    Address                  State     Interface      Time     Interval  Multiplier
    fe80::2ab:44ff:fe08:f801 Up        em1.0          0.900     0.300        3
    fe80::2ab:b1ff:fec4:7200 Up        em0.0          0.900     0.300        3

    2 sessions, 2 clients
    Cumulative transmit rate 6.7 pps, cumulative receive rate 6.7 pps

    lab@VULCAN> 

### BFD Authentication ###

Configuring BFD authentication can be a bit of a pain, but once you
do it a few times, it becomes habit.

On all routers:

    configure exclusive
    edit security authentication-key-chains key-chain BFD_CHAIN key 0
    set secret "$9$WK1Xxdg4ZiHmM8Dkqm3n0BIcrvx7VwgJ"
    set start-time "2000-1-1.00:00:00 +0000"
    top edit protocols ospf3 area 0.0.0.0
    set interface fe-0/0/0.0 bfd-liveness-detection authentication key-chain BFD_CHAIN
    set interface fe-0/0/0.0 bfd-liveness-detection authentication algorithm keyed-sha-1
    set interface fe-0/0/1.0 bfd-liveness-detection authentication key-chain BFD_CHAIN
    set interface fe-0/0/1.0 bfd-liveness-detection authentication algorithm keyed-sha-1
    commit and-quit

#### Verifying BFD ####

Ensure you didn't lose OSPFv3 relationships and that BFD is authenticating:

    show ospf3 neighbor
    show bfd session extensive | match "auth"

`Example`:

    lab@VULCAN> show ospf3 neighbor
    ID               Interface              State     Pri   Dead
    2.2.2.2          em0.0                  Full      128     39
      Neighbor-address fe80::2ab:b1ff:fec4:7200
    3.3.3.3          em1.0                  Full      128     35
      Neighbor-address fe80::2ab:44ff:fe08:f801

    lab@VULCAN> show bfd session extensive | match auth
     Client OSPF realm ipv6-unicast Area 0.0.0.0, TX interval 0.300, RX interval 0.300, Authenticate
     Authentication enabled/active, keychain BFD_CHAIN, algo keyed-sha-1, mode strict
     Client OSPF realm ipv6-unicast Area 0.0.0.0, TX interval 0.300, RX interval 0.300, Authenticate
     Authentication enabled/active, keychain BFD_CHAIN, algo keyed-sha-1, mode strict

    lab@VULCAN>

## Verification ##

Final verification time!

Here are the commands we'll be using:

    show ospf3 neighbor detail
    show ospf3 interface detail
    show ipsec security-associations detail
    show bfd session extensive
    show route 2001:db8::/64 terse
    ping 2001:db8::1 count 1
    ping 2001:db8::2 count 1
    ping 2001:db8::3 count 1
    ping 2001:db8::4 count 1

And an `example`:

    lab@VULCAN> show ospf3 neighbor detail
    ID               Interface              State     Pri   Dead
    2.2.2.2          em0.0                  Full      128     34
      Neighbor-address fe80::2ab:b1ff:fec4:7200
      Area 0.0.0.0, opt 0x13, OSPF3-Intf-Index 2
      DR-ID 0.0.0.0, BDR-ID 0.0.0.0
      Up 00:04:12, adjacent 00:04:07
    3.3.3.3          em1.0                  Full      128     36
      Neighbor-address fe80::2ab:44ff:fe08:f801
      Area 0.0.0.0, opt 0x13, OSPF3-Intf-Index 3
      DR-ID 0.0.0.0, BDR-ID 0.0.0.0
      Up 00:04:14, adjacent 00:04:14

    lab@VULCAN> show ospf3 interface detail
    Interface           State   Area            DR ID           BDR ID          Nbrs
    em0.0               PtToPt  0.0.0.0         0.0.0.0         0.0.0.0            1
      Address fe80::2ab:7dff:fee3:8200, Prefix-length 64
      OSPF3-Intf-index 2, Type P2P, MTU 1500, Cost 1
      Adj count: 1, Router LSA ID: 0
      Hello 10, Dead 40, ReXmit 5, Not Stub
      IPSec SA name: ospf3
      Protection type: None
    em1.0               PtToPt  0.0.0.0         0.0.0.0         0.0.0.0            1
      Address fe80::2ab:7dff:fee3:8201, Prefix-length 64
      OSPF3-Intf-index 3, Type P2P, MTU 1500, Cost 1
      Adj count: 1, Router LSA ID: 0
      Hello 10, Dead 40, ReXmit 5, Not Stub
      IPSec SA name: ospf3
      Protection type: None
    lo0.0               DRother 0.0.0.0         0.0.0.0         0.0.0.0            0
      Address fe80::2ab:7d0f:fce3:8200, Prefix-length 128
      OSPF3-Intf-index 1, Type LAN, MTU 65535, Cost 0
      Adj count: 0, Router LSA ID: -, Passive
      Hello 10, Dead 40, ReXmit 5, Not Stub
      Protection type: None

    lab@VULCAN> show ipsec security-associations detail
    Security association: ospf3

        Direction: inbound, SPI: 256, AUX-SPI: 0
        Mode: transport, Type: manual, State: Installed
        Protocol: AH, Authentication: hmac-sha1-96, Encryption: None
        Anti-replay service: Disabled

        Direction: outbound, SPI: 256, AUX-SPI: 0
        Mode: transport, Type: manual, State: Installed
        Protocol: AH, Authentication: hmac-sha1-96, Encryption: None
        Anti-replay service: Disabled

    lab@VULCAN> show bfd session extensive
                                                      Detect   Transmit
    Address                  State     Interface      Time     Interval  Multiplier
    fe80::2ab:44ff:fe08:f801 Up        em1.0          0.900     0.300        3
     Client OSPF realm ipv6-unicast Area 0.0.0.0, TX interval 0.300, RX interval 0.300, Authenticate
            keychain BFD_CHAIN, algo keyed-sha-1, mode strict
     Session up time 00:04:36, previous down time 00:00:05
     Local diagnostic NbrSignal, remote diagnostic CtlExpire
     Remote state Up, version 1
     Min async interval 0.300, min slow interval 1.000
     Adaptive async TX interval 0.300, RX interval 0.300
     Local min TX interval 0.300, minimum RX interval 0.300, multiplier 3
     Remote min TX interval 0.300, min RX interval 0.300, multiplier 3
     Local discriminator 4, remote discriminator 4
     Echo mode disabled/inactive
     Authentication enabled/active, keychain BFD_CHAIN, algo keyed-sha-1, mode strict

                                                      Detect   Transmit
    Address                  State     Interface      Time     Interval  Multiplier
    fe80::2ab:b1ff:fec4:7200 Up        em0.0          0.900     0.300        3
     Client OSPF realm ipv6-unicast Area 0.0.0.0, TX interval 0.300, RX interval 0.300, Authenticate
            keychain BFD_CHAIN, algo keyed-sha-1, mode strict
     Session up time 00:04:33, previous down time 00:00:06
     Local diagnostic NbrSignal, remote diagnostic CtlExpire
     Remote state Up, version 1
     Min async interval 0.300, min slow interval 1.000
     Adaptive async TX interval 0.300, RX interval 0.300
     Local min TX interval 0.300, minimum RX interval 0.300, multiplier 3
     Remote min TX interval 0.300, min RX interval 0.300, multiplier 3
     Local discriminator 3, remote discriminator 3
     Echo mode disabled/inactive
     Authentication enabled/active, keychain BFD_CHAIN, algo keyed-sha-1, mode strict

    2 sessions, 2 clients
    Cumulative transmit rate 6.7 pps, cumulative receive rate 6.7 pps

    lab@VULCAN> show route 2001:db8::/64 terse

    inet6.0: 9 destinations, 10 routes (9 active, 0 holddown, 0 hidden)
    + = Active Route, - = Last Active, * = Both

    A Destination        P Prf   Metric 1   Metric 2  Next hop         AS path
    * 2001:db8::1/128    D   0                       >lo0.0
    * 2001:db8::2/128    O  10          1            >fe80::2ab:b1ff:fec4:7200
    * 2001:db8::3/128    O  10          1            >fe80::2ab:44ff:fe08:f801
    * 2001:db8::4/128    O  10          2            >fe80::2ab:b1ff:fec4:7200
                                                      fe80::2ab:44ff:fe08:f801

    lab@VULCAN> ping 2001:db8::1 count 1
    PING6(56=40+8+8 bytes) 2001:db8::1 --> 2001:db8::1
    16 bytes from 2001:db8::1, icmp_seq=0 hlim=64 time=0.653 ms

    --- 2001:db8::1 ping6 statistics ---
    1 packets transmitted, 1 packets received, 0% packet loss
    round-trip min/avg/max/std-dev = 0.653/0.653/0.653/0.000 ms

    lab@VULCAN> ping 2001:db8::2 count 1
    PING6(56=40+8+8 bytes) 2001:db8::1 --> 2001:db8::2
    16 bytes from 2001:db8::2, icmp_seq=0 hlim=64 time=2.888 ms

    --- 2001:db8::2 ping6 statistics ---
    1 packets transmitted, 1 packets received, 0% packet loss
    round-trip min/avg/max/std-dev = 2.888/2.888/2.888/0.000 ms

    lab@VULCAN> ping 2001:db8::3 count 1
    PING6(56=40+8+8 bytes) 2001:db8::1 --> 2001:db8::3
    16 bytes from 2001:db8::3, icmp_seq=0 hlim=64 time=1.896 ms

    --- 2001:db8::3 ping6 statistics ---
    1 packets transmitted, 1 packets received, 0% packet loss
    round-trip min/avg/max/std-dev = 1.896/1.896/1.896/0.000 ms

    lab@VULCAN> ping 2001:db8::4 count 1
    PING6(56=40+8+8 bytes) 2001:db8::1 --> 2001:db8::4
    16 bytes from 2001:db8::4, icmp_seq=0 hlim=63 time=3.303 ms

    --- 2001:db8::4 ping6 statistics ---
    1 packets transmitted, 1 packets received, 0% packet loss
    round-trip min/avg/max/std-dev = 3.303/3.303/3.303/0.000 ms

    lab@VULCAN> 

