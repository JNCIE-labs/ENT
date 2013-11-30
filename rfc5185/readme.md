# Preface #
The purpose of this lab is to explore issues related to sub-optimal routing in OSPF domains.  This is not to explore sub-optimal routing due to link failure, but due to normal operation and design.  We will be using the multiarea adjacency feature of OSPF, which is defined in [RFC 5185][1].  There are several options to solving the problem we will outline today, but the multiarea adjacency is the best solution for platforms that support it.
[1]: http://tools.ietf.org/html/rfc5185 "RFC 5185"

## Caveat ##
We will be implementing all of the features in Junos.  Certain vendors do not support it at all or support it in only certain software releases.  It has been supported since Junos 9.2, released 12 August 2008.  It is currently not supported in Cisco's IOS; however, it is present in IOS XE, XR (beginning with 3.4.1), and NXOS.  It is not available in Ericcson/Redback's SeOS, Alcatel-Lucent's TiMOS, or Brocade's NetIron.

# Diagram and Configuration #
We'll be using the `secondary` knob at the `protocols ospf area [x] interface [intf_name]` hierarchy.  Each link is labeled with its speed and subnet.  The devices `jnpr01-srx_vr1` and `jnpr02-srx_vr1` represent customer routers.  The speed of their circuits is irrelevant since they aren't part of the IGP.  Additionally, the PE-CE protocol could be anything, but we'll use static routes for simplicity in this example.  We'll also assume that they're metro Ethernet customers with public IP addresses and have no VPN requirements, so they won't be placed in a VRF.  Therefore, other protocols such as BGP and MPLS will be absent as well.

## Diagram ##
![OSPF Multi-Area Adjacency Diagram][2]

This should be a simple topology to follow and reference.  All interfaces and subnets are listed.  Each router will have a loopback IP address equal to its router number; the virtual-routers will have a varied loopback scheme.  For easy reference, here are the loopback IPs:

<table class="table table-striped">
<thead>
<tr>
<th>Router</th>
<th>Interface</th>
<th>Loopback</th>
</tr>
</thead>
<tbody>
<tr>
<td>jnpr01-srx</td>
<td>lo0.1</td>
<td>1.1.1.1/32</td>
</tr>
<tr>
<td>jnpr02-srx</td>
<td>lo0.2</td>
<td>2.2.2.2/32</td>
</tr>
<tr>
<td>jnpr03-ex</td>
<td>lo0.3</td>
<td>3.3.3.3/32</td>
</tr>
<tr>
<td>jnpr04-m7i</td>
<td>lo0.4</td>
<td>4.4.4.4/32</td>
</tr>
<tr>
<td>jnpr01-srx_vr1</td>
<td>lo0.11</td>
<td>11.11.11.11/32</td>
</tr>
<tr>
<td>jnpr02-srx_vr1</td>
<td>lo0.22</td>
<td>22.22.22.22/32</td>
</tr>
</tbody>
</table>

[2]:

## Configuration ##
Below is the configuration for each device that we'll start with.  We'll then explore the behavior that the configuration and topology exhibit, followed by a discussion of multi-area adjacency and how it solves the problem.

<div class="accordion" id="accordion1">
<div class="accordion-group">
<div class="accordion-heading">
<a class="accordion-toggle" data-toggle="collapse" data-parent="#accordion1" href="#jnpr01-srx">
jnpr01-srx config
</a>
</div>
<div id="jnpr01-srx" class="accordion-body collapse">
<div class="accordion-inner">
<pre class="prettyprint">tylerc@jnpr01-srx# show | compare | no-more
[edit interfaces]
+   lt-0/0/0 {
+       unit 1 {
+           encapsulation ethernet;
+           peer-unit 11;
+           family inet {
+               address 10.1.11.1/24;
+           }
+       }
+       unit 11 {
+           encapsulation ethernet;
+           peer-unit 1;
+           family inet {
+               address 10.1.11.11/24;
+           }
+       }
+   }
+   fe-0/0/2 {
+       unit 0 {
+           family inet {
+               address 10.1.3.1/24;
+           }
+       }
+   }
+   fe-0/0/3 {
+       unit 0 {
+           family inet {
+               address 10.1.4.1/24;
+           }
+       }
+   }
+   lo0 {
+       unit 1 {
+           family inet {
+               address 1.1.1.1/32;
+           }
+       }
+   }
[edit routing-options]
+   static {
+       route 11.11.11.11/32 next-hop 10.1.11.11;
+   }
[edit protocols]
+   ospf {
+       export EXPORT_STATIC;
+       area 0.0.0.0 {
+           interface fe-0/0/3.0 {
+               interface-type p2p;
+               metric 10;
+           }
+           interface lo0.1;
+       }
+       area 0.0.0.7 {
+           interface fe-0/0/2.0 {
+               interface-type p2p;
+           }
+       }
+   }
[edit]
+  policy-options {
+      policy-statement EXPORT_STATIC {
+          from protocol static;
+          then accept;
+      }
+  }
[edit routing-instances]
+   jnpr01-srx_vr1 {
+       instance-type virtual-router;
+       interface lt-0/0/0.11;
+       interface lo0.11;
+   }</pre>
</div>
</div>
</div>
<div class="accordion" id="accordion1">
<div class="accordion-group">
<div class="accordion-heading">
<a class="accordion-toggle" data-toggle="collapse" data-parent="#accordion1" href="#jnpr02-srx">
jnpr02-srx config
</a>
</div>
<div id="jnpr02-srx" class="accordion-body collapse">
<div class="accordion-inner">
<pre class="prettyprint">tylerc@jnpr02-srx# show | compare | no-more
[edit interfaces]
+   lt-0/0/0 {
+       unit 2 {
+           encapsulation ethernet;
+           peer-unit 22;
+           family inet {
+               address 10.2.22.2/24;
+           }
+       }
+       unit 22 {
+           encapsulation ethernet;
+           peer-unit 2;
+           family inet {
+               address 10.2.22.22/24;
+           }
+       }
+   }
+   fe-0/0/2 {
+       unit 0 {
+           family inet {
+               address 10.2.3.2/24;
+           }
+       }
+   }
+   fe-0/0/3 {
+       unit 0 {
+           family inet {
+               address 10.2.4.2/24;
+           }
+       }
+   }
+   lo0 {
+       unit 0;
+       unit 2 {
+           family inet {
+               address 2.2.2.2/32;
+           }
+       }
+       unit 22 {
+           family inet {
+               address 22.22.22.22/32;
+           }
+       }
+   }
[edit routing-options]
+   static {
+       route 22.22.22.22/32 next-hop 10.2.22.22;
+   }
[edit protocols]
+   ospf {
+       export EXPORT_STATIC;
+       area 0.0.0.0 {
+           interface lo0.2;
+           interface fe-0/0/3.0 {
+               interface-type p2p;
+           }
+       }
+       area 0.0.0.7 {
+           interface fe-0/0/2.0 {
+               interface-type p2p;
+           }
+       }
+   }
[edit]
+  policy-options {
+      policy-statement EXPORT_STATIC {
+          from protocol static;
+          then accept;
+      }
+  }
[edit routing-instances]
+   jnpr02-srx_vr1 {
+       instance-type virtual-router;
+       interface lt-0/0/0.22;
+       interface lo0.22;
+   }</pre>
</div>
</div>
</div>
v class="accordion" id="accordion1">
<div class="accordion-group">
<div class="accordion-heading">
<a class="accordion-toggle" data-toggle="collapse" data-parent="#accordion1" href="#jnpr03-ex">
jnpr03-ex config
</a>
</div>
<div id="jnpr03-ex" class="accordion-body collapse">
<div class="accordion-inner">
<pre class="prettyprint">tylerc@jnpr03-ex# show | compare rollback 1 | no-more
[edit interfaces ge-0/0/13]
+    unit 0 {
+        family inet {
+            address 10.2.3.3/24;
+        }
+    }
[edit interfaces ge-0/0/15]
+    unit 0 {
+        family inet {
+            address 10.1.3.3/24;
+        }
+    }
[edit interfaces]
+   lo0 {
+       unit 3 {
+           family inet {
+               address 3.3.3.3/32;
+           }
+       }
+   }
[edit protocols]
+   ospf {
+       area 0.0.0.7 {
+           interface lo0.3;
+           interface ge-0/0/13.0 {
+               interface-type p2p;
+           }
+           interface ge-0/0/15.0 {
+               interface-type p2p;
+           }
+       }
+   }</pre>
</div>
</div>
</div>
<div class="accordion" id="accordion1">
<div class="accordion-group">
<div class="accordion-heading">
<a class="accordion-toggle" data-toggle="collapse" data-parent="#accordion1" href="#jnpr04-m7i">
jnpr04-m7i config
</a>
</div>
<div id="jnpr04-m7i" class="accordion-body collapse">
<div class="accordion-inner">
<pre class="prettyprint">tylerc@jnpr04-m7i# show | compare | no-more
[edit interfaces]
+   fe-0/3/0 {
+       unit 0 {
+           family inet {
+               address 10.2.4.4/24;
+           }
+       }
+   }
+   fe-0/3/1 {
+       unit 0 {
+           family inet {
+               address 10.1.4.4/24;
+           }
+       }
+   }
+   lo0 {
+       unit 4 {
+           family inet {
+               address 4.4.4.4/32;
+           }
+       }
+   }
[edit]
+  protocols {
+      ospf {
+          area 0.0.0.0 {
+              interface lo0.4;
+              interface fe-0/3/0.0 {
+                  interface-type p2p;
+              }
+              interface fe-0/3/1.0 {
+                  interface-type p2p;
+                  metric 10;
+              }
+          }
+      }
+  }</pre>
</div>
</div>
</div>

### Caveat ###
It's worth noting that you would probably never redistribute customer routes into OSPF.  That would be a pretty silly thing to do.  But since BGP does a recursive route lookup, it will rely on the OSPF path.  This is the simplest way to show the effects without adding the overhead of BGP.

Also noteworthy is that you could just as well use a routing policy to influence the BGP next-hop to get around this suboptimal routing situation, but that could quickly become cumbersome and failure-prone.

## Expected Behavior ##
In this case, what we expect and what we want are different.  We want the path to take only the 100M links and bypass the 10M link.  However, we should expect that it will use the 10M link based on our knowledge of OSPF.  If that isn't your expectation, we'll examine why later.  Our verification tool in this example will be `traceroute`.

<div class="accordion" id="accordion2">
<div class="accordion-group">
<div class="accordion-heading">
<a class="accordion-toggle" data-toggle="collapse" data-parent="#accordion2" href="#trace02">
jnpr02-srx_vr1 traceroute
</a>
</div>
<div id="trace02" class="accordion-body collapse">
<div class="accordion-inner">
backbone0 <pre class="prettyprint">tylerc@jnpr02-srx> traceroute 4.4.4.4 routing-instance jnpr02-srx_vr1 source 22.22.22.22
traceroute to 4.4.4.4 (4.4.4.4) from 22.22.22.22, 30 hops max, 40 byte packets
1  10.2.22.2 (10.2.22.2)  6.038 ms  14.141 ms  7.555 ms
2  4.4.4.4 (4.4.4.4)  4.912 ms  5.301 ms  12.634 ms</pre>
</div>
</div>
</div>
<div class="accordion" id="accordion2">
<div class="accordion-group">
<div class="accordion-heading">
<a class="accordion-toggle" data-toggle="collapse" data-parent="#accordion2" href="#trace01">
jnpr01-srx_vr1 traceroute
</a>
</div>
<div id="trace01" class="accordion-body collapse">
<div class="accordion-inner">
<pre class="prettyprint">tylerc@jnpr01-srx> traceroute 4.4.4.4 routing-instance jnpr01-srx_vr1 source 11.11.11.11
traceroute to 4.4.4.4 (4.4.4.4) from 11.11.11.11, 30 hops max, 40 byte packets
1  10.1.11.1 (10.1.11.1)  5.154 ms  3.691 ms  3.214 ms
2  4.4.4.4 (4.4.4.4)  2.872 ms  2.877 ms  2.964 ms</pre>
</div>
</div>
</div>

Here we can clearly see that `jnpr02-srx_vr1` takes the desired path, but `jnpr01-srx_vr1` does not.  `jnpr01-srx_vr1` is taking the 10M link!  Why is this?  It is because of OSPF's own route selection process, which takes place before routes are sent to the RIB.  OSPF's route selection process says: `Intra-Area > Inter-Area > External`.  This means that if a router has multiple paths to a destination, regardless of cost, it will always prefer a route within the area over a route via another area, both of which are preferred over externally-learned routes.  `jnpr01-srx` has an LSA for `4.4.4.4` in its LSDB in both `Area 0` and `Area 7`.  We can verify this with the following:

<div class="accordion" id="accordion3">
<div class="accordion-group">
<div class="accordion-heading">
<a class="accordion-toggle" data-toggle="collapse" data-parent="#accordion3" href="#lsdb">
jnpr01-srx lsdb
</a>
</div>
<div id="lsdb" class="accordion-body collapse">
<div class="accordion-inner">
<pre class="prettyprint">tylerc@jnpr01-srx> show ospf database area 7 detail lsa-id 4.4.4.4

OSPF database, Area 0.0.0.7
Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len
Summary *4.4.4.4          1.1.1.1          0x80000001  1310  0x22 0x1708  28
  mask 255.255.255.255
  Topology default (ID 0) -> Metric: 10
Summary  4.4.4.4          2.2.2.2          0x80000001  1234  0x22 0x9e85  28
  mask 255.255.255.255
  Topology default (ID 0) -> Metric: 1

tylerc@jnpr01-srx> show ospf database area 0 detail lsa-id 4.4.4.4 | match 4.4.4.4
Router   4.4.4.4          4.4.4.4          0x80000004  1248  0x22 0x91c9  84
  id 4.4.4.4, data 255.255.255.255, Type Stub (3)</pre>
</div>
</div>
</div>

We can see now that the LSDB on `jnpr01-srx` contains a Type 1 and a Type 3 LSA for the `4.4.4.4/32` route.  Due to OSPF's route selection criteria, it's going to use the Type 1 LSA, which happens to say that the next-hop is via the 10M link.

# Fixing the Problem #
So how do we fix this?  As I said in the preface, there are a variety of options.  BGP next-hop manipulation _could_ work but would be _very_ messy.  Redesigning the OSPF network is another option, as is a single-area network.  Virtual links might be a possibility except where they're not (which ideally should be always but practically could be stub areas).  The RFC 5185 Multi-Area Adjacency is the best solution in this example.  To accomplish our goals, we will use multi-area adjacency to extend area 0.

## Extending the Backbone ##
To fix the problem, we must allow for the propogation of Type 1 LSAs from `jnpr04-m7i` to `jnpr01-srx` via all available paths to `jnpr01-srx`.  This ensures a more accurate "shortest path" calculation.  You might consider additional logical interfaces and placing them in area 0, but this may not be a viable option of you are performing this as an emergency or if the free IP addresses simply do not exist or would otherwise introduce additional complexity.  Multi-area adjacency it is!


<div class="accordion" id="accordion4">
<div class="accordion-group">
<div class="accordion-heading">
<a class="accordion-toggle" data-toggle="collapse" data-parent="#accordion4" href="#jnpr01_changes">
jnpr01-srx multi-area config
</a>
</div>
<div id="jnpr01_changes" class="accordion-body collapse">
<div class="accordion-inner">
<pre class="prettyprint">tylerc@jnpr01-srx# show | compare
[edit protocols ospf area 0.0.0.0]
interface fe-0/0/3.0 { ... }
+     interface fe-0/0/2.0 {
+         interface-type p2p;
+         secondary;
+     }</pre>
</div>
</div>
</div>
<div class="accordion" id="accordion4">
<div class="accordion-group">
<div class="accordion-heading">
<a class="accordion-toggle" data-toggle="collapse" data-parent="#accordion4" href="#jnpr02_changes">
jnpr02-srx multi-area config
</a>
</div>
<div id="jnpr02_changes" class="accordion-body collapse">
<div class="accordion-inner">
<pre class="prettyprint">tylerc@jnpr02-srx# show | compare
[edit protocols ospf area 0.0.0.0]
interface fe-0/0/3.0 { ... }
+     interface fe-0/0/2.0 {
+         interface-type p2p;
+         secondary;
+     }</pre>
</div>
</div>
</div>
<div class="accordion" id="accordion4">
<div class="accordion-group">
<div class="accordion-heading">
<a class="accordion-toggle" data-toggle="collapse" data-parent="#accordion4" href="#jnpr03_changes">
jnpr03-ex multi-area config
</a>
</div>
<div id="jnpr03_changes" class="accordion-body collapse">
<div class="accordion-inner">
<pre class="prettyprint">tylerc@jnpr03-ex# show | compare
[edit protocols ospf]
area 0.0.0.7 { ... }
+    area 0.0.0.0 {
+        interface ge-0/0/13.0 {
+            interface-type p2p;
+            secondary;
+        }
+        interface ge-0/0/15.0 {
+            interface-type p2p;
+            secondary;
+        }
+    }</pre>
</div>
</div>
</div>

## Verifying Changes ##
Now let's take a look at a traceroute from the perspective of `jnpr01-srx_vr1` and see if it traverse `jnpr03-ex`, skipping the suboptimal 10M circuit.

<div class="accordion" id="accordion5">
<div class="accordion-group">
<div class="accordion-heading">
<a class="accordion-toggle" data-toggle="collapse" data-parent="#accordion5" href="#jnpr01_fixed">
jnpr01-srx_vr1 traceroute
</a>
</div>
<div id="jnpr01_fixed" class="accordion-body collapse">
<div class="accordion-inner">
<pre class="prettyprint">tylerc@jnpr01-srx> traceroute 4.4.4.4 routing-instance jnpr01-srx_vr1 source 11.11.11.11
traceroute to 4.4.4.4 (4.4.4.4) from 11.11.11.11, 30 hops max, 40 byte packets
1  10.1.11.1 (10.1.11.1)  4.871 ms  2.735 ms  3.412 ms
2  10.1.3.3 (10.1.3.3)  3.131 ms  3.275 ms  3.186 ms
3  10.2.3.2 (10.2.3.2)  2.914 ms  4.324 ms  4.961 ms
4  4.4.4.4 (4.4.4.4)  3.044 ms  2.754 ms  3.031 ms

tylerc@jnpr01-srx></pre>
</div>
</div>
</div>

And all is right with the world.  Just to be thorough, lets take a look at the LSDB on `jnpr01-srx`.

<div class="accordion" id="accordion6">
<div class="accordion-group">
<div class="accordion-heading">
<a class="accordion-toggle" data-toggle="collapse" data-parent="#accordion5" href="#jnpr01_fixed">
jnpr01-srx_vr1 traceroute
</a>
</div>
<div id="jnpr01_fixed" class="accordion-body collapse">
<div class="accordion-inner">
<pre class="prettyprint">tylerc@jnpr01-srx> show ospf database router detail | match "Router|4.4.4.4"
Router  *1.1.1.1          1.1.1.1          0x8000003a   460  0x22 0xdf58  60
  id 4.4.4.4, data 10.1.4.1, Type PointToPoint (1)
    Type: PointToPoint, Node ID: 4.4.4.4
Router   2.2.2.2          2.2.2.2          0x80000041   465  0x22 0x6ab5  72
  id 4.4.4.4, data 10.2.4.2, Type PointToPoint (1)
    Type: PointToPoint, Node ID: 4.4.4.4
Router   3.3.3.3          3.3.3.3          0x8000001b   731  0x22 0xfc4c  48
Router   4.4.4.4          4.4.4.4          0x8000003b   461  0x22 0x2301  84
  id 4.4.4.4, data 255.255.255.255, Type Stub (3)
Router  *1.1.1.1          1.1.1.1          0x80000035   653  0x22 0xfca9  60
Router   2.2.2.2          2.2.2.2          0x80000033  2035  0x22 0xcce4  48
Router   3.3.3.3          3.3.3.3          0x8000002a   476  0x22 0xa5b2  84

tylerc@jnpr01-srx> show ospf route 4.4.4.4
Topology default Route Table:

Prefix             Path  Route      NH       Metric NextHop       Nexthop
                   Type  Type       Type            Interface     Address/LSP
4.4.4.4            Intra Router     IP            3 fe-0/0/2.0    10.1.3.3
4.4.4.4/32         Intra Network    IP            3 fe-0/0/2.0    10.1.3.3</pre>
</div>
</div>
</div>

As expected, we can see the Type 1 LSA from `jnpr02-srx` has been received.

# Conclusion #
I can't stress enough that there are several ways to fix this issue.  This was an issue that we recently discovered in our production network; unfortuantely, because IOS doesn't support RFC 5185, we couldn't implement this solution.  Extending Area 0 by changin the area a link participated in was the ultimate decision, though it may not have been the best.  It accomplished (more or less) the same goals.

