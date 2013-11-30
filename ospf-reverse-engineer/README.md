### Reverse Engineering an OSPF Topology ###

This lab is designed to test your knowledge of OSPF LSAs.

You should be able to examine the OSPF database from the two routers below and
properly reconstruct the OSPF topology, including all links, IP addresses,
subnet masks, and area boundaries.

> There is no configuration exercise in this lab.  This lab exists solely
> to test your knowledge of OSPF and your ability to interpret the LSDB.

[R2][2]:

    root@JNCIE-R2> show ospf database extensive | no-more

        OSPF database, Area 0.0.0.0
     Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len
    Router  *2.2.2.2          2.2.2.2          0x8000000c   747  0x22 0x858d  72
      bits 0x1, link count 4
      id 4.4.4.4, data 192.168.0.4, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.4, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      id 3.3.3.3, data 192.168.0.6, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.6, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      Topology default (ID 0)
        Type: PointToPoint, Node ID: 3.3.3.3
          Metric: 1, Bidirectional
        Type: PointToPoint, Node ID: 4.4.4.4
          Metric: 1, Bidirectional
      Gen timer 00:08:10
      Aging timer 00:47:32
      Installed 00:12:27 ago, expires in 00:47:33, sent 00:12:25 ago
      Last changed 00:36:11 ago, Change count: 3, Ours
    Router   3.3.3.3          3.3.3.3          0x8000000e   744  0x22 0xffcd 108
      bits 0x1, link count 7
      id 4.4.4.4, data 192.168.0.8, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.8, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      id 5.5.5.5, data 192.168.0.10, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.10, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      id 2.2.2.2, data 192.168.0.7, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.6, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      id 3.3.3.3, data 255.255.255.255, Type Stub (3)
        Topology count: 0, Default metric: 0
      Topology default (ID 0)
        Type: PointToPoint, Node ID: 2.2.2.2
          Metric: 1, Bidirectional
        Type: PointToPoint, Node ID: 5.5.5.5
          Metric: 1, Bidirectional
        Type: PointToPoint, Node ID: 4.4.4.4
          Metric: 1, Bidirectional
      Aging timer 00:47:36
      Installed 00:12:19 ago, expires in 00:47:36, sent 00:12:17 ago
      Last changed 00:31:26 ago, Change count: 5
    Router   4.4.4.4          4.4.4.4          0x80000004  1894  0x22 0x1eb1 108
      bits 0x0, link count 7
      id 3.3.3.3, data 192.168.0.9, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.8, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      id 2.2.2.2, data 192.168.0.5, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.4, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      id 5.5.5.5, data 192.168.0.12, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.12, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      id 4.4.4.4, data 255.255.255.255, Type Stub (3)
        Topology count: 0, Default metric: 0
      Topology default (ID 0)
        Type: PointToPoint, Node ID: 5.5.5.5
          Metric: 1, Bidirectional
        Type: PointToPoint, Node ID: 2.2.2.2
          Metric: 1, Bidirectional
        Type: PointToPoint, Node ID: 3.3.3.3
          Metric: 1, Bidirectional
      Aging timer 00:28:25
      Installed 00:31:31 ago, expires in 00:28:26, sent 00:31:31 ago
      Last changed 00:31:31 ago, Change count: 3
    Router   5.5.5.5          5.5.5.5          0x80000004   344  0x22 0x6262  84
      bits 0x1, link count 5
      id 3.3.3.3, data 192.168.0.11, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.10, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      id 4.4.4.4, data 192.168.0.13, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.12, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      id 5.5.5.5, data 255.255.255.255, Type Stub (3)
        Topology count: 0, Default metric: 0
      Topology default (ID 0)
        Type: PointToPoint, Node ID: 4.4.4.4
          Metric: 1, Bidirectional
        Type: PointToPoint, Node ID: 3.3.3.3
          Metric: 1, Bidirectional
      Aging timer 00:54:16
      Installed 00:05:37 ago, expires in 00:54:16, sent 00:05:35 ago
      Last changed 00:31:21 ago, Change count: 2
    Summary *1.1.1.1          2.2.2.2          0x80000002   558  0x22 0x2708  28
      mask 255.255.255.255
      Topology default (ID 0) -> Metric: 1
      Gen timer 00:35:16
      Aging timer 00:50:42
      Installed 00:09:18 ago, expires in 00:50:42, sent 00:09:15 ago
      Last changed 00:09:23 ago, Change count: 1, Ours
    Summary  1.1.1.1          3.3.3.3          0x80000001   575  0x22 0xb21   28
      mask 255.255.255.255
      Topology default (ID 0) -> Metric: 1
      Aging timer 00:50:25
      Installed 00:09:33 ago, expires in 00:50:25, sent 00:09:33 ago
      Last changed 00:09:33 ago, Change count: 1
    Summary *2.2.2.2          2.2.2.2          0x80000005   669  0x22 0xe840  28
      mask 255.255.255.255
      Topology default (ID 0) -> Metric: 0
      Gen timer 00:13:35
      Aging timer 00:48:50
      Installed 00:11:09 ago, expires in 00:48:51, sent 00:11:05 ago
      Last changed 00:40:55 ago, Change count: 1, Ours
    Summary  2.2.2.2          3.3.3.3          0x80000001   564  0x22 0xe640  28
      mask 255.255.255.255
      Topology default (ID 0) -> Metric: 2
      Aging timer 00:50:35
      Installed 00:09:23 ago, expires in 00:50:36, sent 00:09:23 ago
      Last changed 00:09:23 ago, Change count: 1
    Summary *192.168.0.0      2.2.2.2          0x80000005   158  0x22 0x9730  28
      mask 255.255.255.254
      Topology default (ID 0) -> Metric: 1
      Gen timer 00:47:21
      Aging timer 00:57:21
      Installed 00:02:38 ago, expires in 00:57:22, sent 00:02:35 ago
      Last changed 00:40:55 ago, Change count: 1, Ours
    Summary  192.168.0.0      3.3.3.3          0x80000001   575  0x22 0x8b3b  28
      mask 255.255.255.254
      Topology default (ID 0) -> Metric: 2
      Aging timer 00:50:25
      Installed 00:09:33 ago, expires in 00:50:25, sent 00:09:33 ago
      Last changed 00:09:33 ago, Change count: 1
    Summary *192.168.0.2      2.2.2.2          0x80000002   558  0x22 0x9334  28
      mask 255.255.255.254
      Topology default (ID 0) -> Metric: 2
      Gen timer 00:40:42
      Aging timer 00:50:42
      Installed 00:09:18 ago, expires in 00:50:42, sent 00:09:15 ago
      Last changed 00:09:23 ago, Change count: 1, Ours
    Summary  192.168.0.2      3.3.3.3          0x80000004    70  0x22 0x675b  28
      mask 255.255.255.254
      Topology default (ID 0) -> Metric: 1
      Aging timer 00:58:49
      Installed 00:01:07 ago, expires in 00:58:50, sent 00:01:05 ago
      Last changed 00:37:01 ago, Change count: 1
    Summary  192.168.0.14     5.5.5.5          0x80000003   656  0x22 0xb4fa  28
      mask 255.255.255.254
      Topology default (ID 0) -> Metric: 1
      Aging timer 00:49:03
      Installed 00:10:50 ago, expires in 00:49:04, sent 00:10:47 ago
      Last changed 00:31:26 ago, Change count: 1
    Summary  192.168.0.16     5.5.5.5          0x80000003   502  0x22 0xa00d  28
      mask 255.255.255.254
      Topology default (ID 0) -> Metric: 1
      Aging timer 00:51:37
      Installed 00:08:13 ago, expires in 00:51:38, sent 00:08:11 ago
      Last changed 00:31:26 ago, Change count: 1
    Summary  192.168.0.18     5.5.5.5          0x80000002  1782  0x22 0x9813  28
      mask 255.255.255.254
      Topology default (ID 0) -> Metric: 2
      Aging timer 00:30:17
      Installed 00:29:35 ago, expires in 00:30:18, sent 00:29:33 ago
      Last changed 00:30:04 ago, Change count: 1
    Summary *192.168.1.0      2.2.2.2          0x80000005   566  0x22 0x9233  28
      mask 255.255.255.0
      Topology default (ID 0) -> Metric: 1
      Gen timer 00:24:26
      Aging timer 00:50:33
      Installed 00:09:26 ago, expires in 00:50:34, sent 00:09:24 ago
      Last changed 00:40:55 ago, Change count: 1, Ours
    Summary  192.168.1.0      3.3.3.3          0x80000002   565  0x22 0x843f  28
      mask 255.255.255.0
      Topology default (ID 0) -> Metric: 2
      Aging timer 00:50:34
      Installed 00:09:21 ago, expires in 00:50:35, sent 00:09:19 ago
      Last changed 00:09:33 ago, Change count: 1

        OSPF database, Area 0.0.0.73
     Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len
    Router   1.1.1.1          1.1.1.1          0x80000005   560  0x20 0xb53e  96
      bits 0x0, link count 6
      id 2.2.2.2, data 192.168.0.0, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.0, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      id 3.3.3.3, data 192.168.0.2, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.2, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      id 192.168.1.79, data 192.168.1.1, Type Transit (2)
        Topology count: 0, Default metric: 1
      id 1.1.1.1, data 255.255.255.255, Type Stub (3)
        Topology count: 0, Default metric: 0
      Topology default (ID 0)
        Type: Transit, Node ID: 192.168.1.79
          Metric: 1, Bidirectional
        Type: PointToPoint, Node ID: 3.3.3.3
          Metric: 1, Bidirectional
        Type: PointToPoint, Node ID: 2.2.2.2
          Metric: 1, Bidirectional
      Aging timer 00:50:39
      Installed 00:09:19 ago, expires in 00:50:40, sent 00:09:19 ago
      Last changed 00:09:19 ago, Change count: 3
    Router  *2.2.2.2          2.2.2.2          0x8000000d   559  0x20 0x6332  72
      bits 0x1, link count 4
      id 1.1.1.1, data 192.168.0.1, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.0, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      id 192.168.1.79, data 192.168.1.79, Type Transit (2)
        Topology count: 0, Default metric: 1
      id 2.2.2.2, data 255.255.255.255, Type Stub (3)
        Topology count: 0, Default metric: 0
      Topology default (ID 0)
        Type: Transit, Node ID: 192.168.1.79
          Metric: 1, Bidirectional
        Type: PointToPoint, Node ID: 1.1.1.1
          Metric: 1, Bidirectional
      Gen timer 00:29:51
      Aging timer 00:50:40
      Installed 00:09:19 ago, expires in 00:50:41, sent 00:09:19 ago
      Last changed 00:09:19 ago, Change count: 3, Ours
    Router   3.3.3.3          3.3.3.3          0x80000009   574  0x20 0xa088  48
      bits 0x1, link count 2
      id 1.1.1.1, data 192.168.0.3, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.2, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      Topology default (ID 0)
        Type: PointToPoint, Node ID: 1.1.1.1
          Metric: 1, Bidirectional
      Aging timer 00:50:25
      Installed 00:09:26 ago, expires in 00:50:26
      Last changed 00:09:26 ago, Change count: 1
    Network *192.168.1.79     2.2.2.2          0x80000001   566  0x20 0x224c  32
      mask 255.255.255.0
      attached router 2.2.2.2
      attached router 1.1.1.1
      Topology default (ID 0)
        Type: Transit, Node ID: 1.1.1.1
          Metric: 0, Bidirectional
        Type: Transit, Node ID: 2.2.2.2
          Metric: 0, Bidirectional
      Gen timer 00:19:00
      Aging timer 00:50:33
      Installed 00:09:26 ago, expires in 00:50:34, sent 00:09:26 ago
      Last changed 00:09:26 ago, Change count: 1, Ours
    Summary *0.0.0.0          2.2.2.2          0x80000001   852  0x20 0x577b  28
      mask 0.0.0.0
      Topology default (ID 0) -> Metric: 100
      Gen timer 00:02:44
      Aging timer 00:45:48
      Installed 00:14:12 ago, expires in 00:45:48, sent 00:09:27 ago
      Last changed 00:14:12 ago, Change count: 1, Ours
    Summary  0.0.0.0          3.3.3.3          0x80000001   834  0x20 0x3995  28
      mask 0.0.0.0
      Topology default (ID 0) -> Metric: 100
      Aging timer 00:46:05
      Installed 00:09:26 ago, expires in 00:46:06
      Last changed 00:09:26 ago, Change count: 1

    root@JNCIE-R2>

[R5][5]:

    root@JNCIE-R5> show ospf database extensive | no-more

        OSPF database, Area 0.0.0.0
     Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len
    Router   2.2.2.2          2.2.2.2          0x8000000c   925  0x22 0x858d  72
      bits 0x1, link count 4
      id 4.4.4.4, data 192.168.0.4, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.4, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      id 3.3.3.3, data 192.168.0.6, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.6, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      Topology default (ID 0)
        Type: PointToPoint, Node ID: 3.3.3.3
          Metric: 1, Bidirectional
        Type: PointToPoint, Node ID: 4.4.4.4
          Metric: 1, Bidirectional
      Aging timer 00:44:35
      Installed 00:15:19 ago, expires in 00:44:35, sent 00:15:16 ago
      Last changed 00:34:29 ago, Change count: 1
    Router   3.3.3.3          3.3.3.3          0x8000000e   919  0x22 0xffcd 108
      bits 0x1, link count 7
      id 4.4.4.4, data 192.168.0.8, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.8, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      id 5.5.5.5, data 192.168.0.10, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.10, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      id 2.2.2.2, data 192.168.0.7, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.6, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      id 3.3.3.3, data 255.255.255.255, Type Stub (3)
        Topology count: 0, Default metric: 0
      Topology default (ID 0)
        Type: PointToPoint, Node ID: 2.2.2.2
          Metric: 1, Bidirectional
        Type: PointToPoint, Node ID: 5.5.5.5
          Metric: 1, Bidirectional
        Type: PointToPoint, Node ID: 4.4.4.4
          Metric: 1, Bidirectional
      Aging timer 00:44:40
      Installed 00:15:14 ago, expires in 00:44:41, sent 00:15:12 ago
      Last changed 00:34:21 ago, Change count: 2
    Router   4.4.4.4          4.4.4.4          0x80000004  2070  0x22 0x1eb1 108
      bits 0x0, link count 7
      id 3.3.3.3, data 192.168.0.9, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.8, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      id 2.2.2.2, data 192.168.0.5, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.4, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      id 5.5.5.5, data 192.168.0.12, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.12, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      id 4.4.4.4, data 255.255.255.255, Type Stub (3)
        Topology count: 0, Default metric: 0
      Topology default (ID 0)
        Type: PointToPoint, Node ID: 5.5.5.5
          Metric: 1, Bidirectional
        Type: PointToPoint, Node ID: 2.2.2.2
          Metric: 1, Bidirectional
        Type: PointToPoint, Node ID: 3.3.3.3
          Metric: 1, Bidirectional
      Aging timer 00:25:29
      Installed 00:34:27 ago, expires in 00:25:30
      Last changed 00:34:27 ago, Change count: 2
    Router  *5.5.5.5          5.5.5.5          0x80000004   518  0x22 0x6262  84
      bits 0x1, link count 5
      id 3.3.3.3, data 192.168.0.11, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.10, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      id 4.4.4.4, data 192.168.0.13, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.12, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      id 5.5.5.5, data 255.255.255.255, Type Stub (3)
        Topology count: 0, Default metric: 0
      Topology default (ID 0)
        Type: PointToPoint, Node ID: 4.4.4.4
          Metric: 1, Bidirectional
        Type: PointToPoint, Node ID: 3.3.3.3
          Metric: 1, Bidirectional
      Gen timer 00:41:22
      Aging timer 00:51:22
      Installed 00:08:38 ago, expires in 00:51:22, sent 00:08:29 ago
      Last changed 00:34:21 ago, Change count: 2, Ours
    Summary  1.1.1.1          2.2.2.2          0x80000002   736  0x22 0x2708  28
      mask 255.255.255.255
      Topology default (ID 0) -> Metric: 1
      Aging timer 00:47:44
      Installed 00:12:09 ago, expires in 00:47:44, sent 00:12:07 ago
      Last changed 00:12:19 ago, Change count: 1
    Summary  1.1.1.1          3.3.3.3          0x80000001   749  0x22 0xb21   28
      mask 255.255.255.255
      Topology default (ID 0) -> Metric: 1
      Aging timer 00:47:30
      Installed 00:12:28 ago, expires in 00:47:31, sent 00:12:28 ago
      Last changed 00:12:28 ago, Change count: 1
    Summary  2.2.2.2          2.2.2.2          0x80000005   847  0x22 0xe840  28
      mask 255.255.255.255
      Topology default (ID 0) -> Metric: 0
      Aging timer 00:45:53
      Installed 00:13:59 ago, expires in 00:45:53, sent 00:13:57 ago
      Last changed 00:34:29 ago, Change count: 1
    Summary  2.2.2.2          3.3.3.3          0x80000001   740  0x22 0xe640  28
      mask 255.255.255.255
      Topology default (ID 0) -> Metric: 2
      Aging timer 00:47:39
      Installed 00:12:19 ago, expires in 00:47:40, sent 00:12:19 ago
      Last changed 00:12:19 ago, Change count: 1
    Summary  192.168.0.0      2.2.2.2          0x80000005   335  0x22 0x9730  28
      mask 255.255.255.254
      Topology default (ID 0) -> Metric: 1
      Aging timer 00:54:24
      Installed 00:05:28 ago, expires in 00:54:25, sent 00:05:26 ago
      Last changed 00:34:29 ago, Change count: 1
    Summary  192.168.0.0      3.3.3.3          0x80000001   749  0x22 0x8b3b  28
      mask 255.255.255.254
      Topology default (ID 0) -> Metric: 2
      Aging timer 00:47:30
      Installed 00:12:28 ago, expires in 00:47:31, sent 00:12:28 ago
      Last changed 00:12:28 ago, Change count: 1
    Summary  192.168.0.2      2.2.2.2          0x80000002   736  0x22 0x9334  28
      mask 255.255.255.254
      Topology default (ID 0) -> Metric: 2
      Aging timer 00:47:44
      Installed 00:12:09 ago, expires in 00:47:44, sent 00:12:07 ago
      Last changed 00:12:19 ago, Change count: 1
    Summary  192.168.0.2      3.3.3.3          0x80000004   244  0x22 0x675b  28
      mask 255.255.255.254
      Topology default (ID 0) -> Metric: 1
      Aging timer 00:55:56
      Installed 00:04:01 ago, expires in 00:55:56, sent 00:03:59 ago
      Last changed 00:34:29 ago, Change count: 1
    Summary *192.168.0.14     5.5.5.5          0x80000003   830  0x22 0xb4fa  28
      mask 255.255.255.254
      Topology default (ID 0) -> Metric: 1
      Gen timer 00:33:04
      Aging timer 00:46:09
      Installed 00:13:50 ago, expires in 00:46:10, sent 00:13:48 ago
      Last changed 00:34:25 ago, Change count: 1, Ours
    Summary *192.168.0.16     5.5.5.5          0x80000003   675  0x22 0xa00d  28
      mask 255.255.255.254
      Topology default (ID 0) -> Metric: 1
      Gen timer 00:38:38
      Aging timer 00:48:45
      Installed 00:11:15 ago, expires in 00:48:45, sent 00:11:11 ago
      Last changed 00:34:25 ago, Change count: 1, Ours
    Summary *192.168.0.18     5.5.5.5          0x80000002  1956  0x22 0x9813  28
      mask 255.255.255.254
      Topology default (ID 0) -> Metric: 2
      Gen timer 00:16:23
      Aging timer 00:27:23
      Installed 00:32:36 ago, expires in 00:27:24, sent 00:32:28 ago
      Last changed 00:33:02 ago, Change count: 1, Ours
    Summary  192.168.1.0      2.2.2.2          0x80000005   744  0x22 0x9233  28
      mask 255.255.255.0
      Topology default (ID 0) -> Metric: 1
      Aging timer 00:47:36
      Installed 00:12:18 ago, expires in 00:47:36, sent 00:12:15 ago
      Last changed 00:34:29 ago, Change count: 1
    Summary  192.168.1.0      3.3.3.3          0x80000002   739  0x22 0x843f  28
      mask 255.255.255.0
      Topology default (ID 0) -> Metric: 2
      Aging timer 00:47:41
      Installed 00:12:17 ago, expires in 00:47:41, sent 00:12:15 ago
      Last changed 00:12:28 ago, Change count: 1

        OSPF database, Area 0.0.0.41
     Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len
    Router  *5.5.5.5          5.5.5.5          0x80000004  1964  0x22 0xb30f  72
      bits 0x1, link count 4
      id 6.6.6.6, data 192.168.0.14, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.14, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      id 7.7.7.7, data 192.168.0.16, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.16, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      Topology default (ID 0)
        Type: PointToPoint, Node ID: 7.7.7.7
          Metric: 1, Bidirectional
        Type: PointToPoint, Node ID: 6.6.6.6
          Metric: 1, Bidirectional
      Gen timer 00:13:36
      Aging timer 00:27:15
      Installed 00:32:44 ago, expires in 00:27:16, sent 00:32:41 ago
      Last changed 00:32:44 ago, Change count: 3, Ours
    Router   6.6.6.6          6.6.6.6          0x80000003  1961  0x22 0x912a  72
      bits 0x0, link count 4
      id 7.7.7.7, data 192.168.0.18, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.18, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      id 5.5.5.5, data 192.168.0.15, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.14, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      Topology default (ID 0)
        Type: PointToPoint, Node ID: 5.5.5.5
          Metric: 1, Bidirectional
        Type: PointToPoint, Node ID: 7.7.7.7
          Metric: 1, Bidirectional
      Aging timer 00:27:18
      Installed 00:32:37 ago, expires in 00:27:19, sent 00:32:35 ago
      Last changed 00:32:37 ago, Change count: 2
    Router   7.7.7.7          7.7.7.7          0x80000003  1970  0x22 0xd9d8  72
      bits 0x0, link count 4
      id 6.6.6.6, data 192.168.0.19, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.18, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      id 5.5.5.5, data 192.168.0.17, Type PointToPoint (1)
        Topology count: 0, Default metric: 1
      id 192.168.0.16, data 255.255.255.254, Type Stub (3)
        Topology count: 0, Default metric: 1
      Topology default (ID 0)
        Type: PointToPoint, Node ID: 5.5.5.5
          Metric: 1, Bidirectional
        Type: PointToPoint, Node ID: 6.6.6.6
          Metric: 1, Bidirectional
      Aging timer 00:27:09
      Installed 00:32:47 ago, expires in 00:27:10, sent 00:32:39 ago
      Last changed 00:32:47 ago, Change count: 2
    Summary *1.1.1.1          5.5.5.5          0x80000001   748  0x22 0xd84a  28
      mask 255.255.255.255
      Topology default (ID 0) -> Metric: 2
      Gen timer 00:35:51
      Aging timer 00:47:31
      Installed 00:12:28 ago, expires in 00:47:32, sent 00:12:28 ago
      Last changed 00:12:28 ago, Change count: 1, Ours
    Summary *2.2.2.2          5.5.5.5          0x80000003   348  0x22 0xa676  28
      mask 255.255.255.255
      Topology default (ID 0) -> Metric: 2
      Gen timer 00:44:11
      Aging timer 00:54:11
      Installed 00:05:48 ago, expires in 00:54:12, sent 00:05:46 ago
      Last changed 00:34:26 ago, Change count: 1, Ours
    Summary *3.3.3.3          5.5.5.5          0x80000003   183  0x22 0x6eab  28
      mask 255.255.255.255
      Topology default (ID 0) -> Metric: 1
      Gen timer 00:46:56
      Aging timer 00:56:56
      Installed 00:03:03 ago, expires in 00:56:57, sent 00:03:00 ago
      Last changed 00:34:21 ago, Change count: 2, Ours
    Summary *4.4.4.4          5.5.5.5          0x80000002  1613  0x22 0x42d4  28
      mask 255.255.255.255
      Topology default (ID 0) -> Metric: 1
      Gen timer 00:19:10
      Aging timer 00:33:06
      Installed 00:26:53 ago, expires in 00:33:07, sent 00:26:51 ago
      Last changed 00:34:26 ago, Change count: 1, Ours
    Summary *5.5.5.5          5.5.5.5          0x80000003  1299  0x22 0x80b   28
      mask 255.255.255.255
      Topology default (ID 0) -> Metric: 0
      Gen timer 00:24:43
      Aging timer 00:38:20
      Installed 00:21:39 ago, expires in 00:38:21, sent 00:21:35 ago
      Last changed 00:34:26 ago, Change count: 1, Ours
    Summary *192.168.0.0      5.5.5.5          0x80000003    16  0x22 0x5566  28
      mask 255.255.255.254
      Topology default (ID 0) -> Metric: 3
      Gen timer 00:49:43
      Aging timer 00:59:43
      Installed 00:00:16 ago, expires in 00:59:44, sent 00:00:13 ago
      Last changed 00:34:26 ago, Change count: 1, Ours
    Summary *192.168.0.2      5.5.5.5          0x80000002  2061  0x22 0x3982  28
      mask 255.255.255.254
      Topology default (ID 0) -> Metric: 2
      Gen timer 00:02:29
      Aging timer 00:25:39
      Installed 00:34:21 ago, expires in 00:25:39, sent 00:33:10 ago
      Last changed 00:34:21 ago, Change count: 2, Ours
    Summary *192.168.0.4      5.5.5.5          0x80000002  1455  0x22 0x2594  28
      mask 255.255.255.254
      Topology default (ID 0) -> Metric: 2
      Gen timer 00:21:56
      Aging timer 00:35:44
      Installed 00:24:15 ago, expires in 00:35:45, sent 00:24:11 ago
      Last changed 00:34:26 ago, Change count: 1, Ours
    Summary *192.168.0.6      5.5.5.5          0x80000002  2061  0x22 0x11a6  28
      mask 255.255.255.254
      Topology default (ID 0) -> Metric: 2
      Gen timer 00:05:15
      Aging timer 00:25:39
      Installed 00:34:21 ago, expires in 00:25:39, sent 00:33:10 ago
      Last changed 00:34:21 ago, Change count: 2, Ours
    Summary *192.168.0.8      5.5.5.5          0x80000002  2061  0x22 0xfcb8  28
      mask 255.255.255.254
      Topology default (ID 0) -> Metric: 2
      Gen timer 00:08:02
      Aging timer 00:25:39
      Installed 00:34:21 ago, expires in 00:25:39, sent 00:33:10 ago
      Last changed 00:34:26 ago, Change count: 1, Ours
    Summary *192.168.0.10     5.5.5.5          0x80000003  1143  0x22 0xdcd6  28
      mask 255.255.255.254
      Topology default (ID 0) -> Metric: 1
      Gen timer 00:27:30
      Aging timer 00:40:57
      Installed 00:19:03 ago, expires in 00:40:57, sent 00:19:00 ago
      Last changed 00:34:26 ago, Change count: 1, Ours
    Summary *192.168.0.12     5.5.5.5          0x80000003   986  0x22 0xc8e8  28
      mask 255.255.255.254
      Topology default (ID 0) -> Metric: 1
      Gen timer 00:30:17
      Aging timer 00:43:34
      Installed 00:16:26 ago, expires in 00:43:34, sent 00:16:23 ago
      Last changed 00:34:26 ago, Change count: 1, Ours
    Summary *192.168.1.0      5.5.5.5          0x80000002  2061  0x22 0x5268  28
      mask 255.255.255.0
      Topology default (ID 0) -> Metric: 3
      Gen timer 00:10:49
      Aging timer 00:25:39
      Installed 00:34:21 ago, expires in 00:25:39, sent 00:33:10 ago
      Last changed 00:34:26 ago, Change count: 1, Ours

    root@JNCIE-R5>

[2]: #
[5]: #
