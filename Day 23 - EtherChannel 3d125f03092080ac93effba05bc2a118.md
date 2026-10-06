# Day 23 - EtherChannel

## Configure Layer 2 EtherChannel between ASW1 and DSW1

### ASW1 LACP

1. Enable global configuration mode and enter the g0/1 - 2 interfaces 
2. Create EtherChannel  in active mode using LACP 

```jsx
ASW1(config-if-range)#channel-group 1 mode ?
  active     Enable LACP unconditionally
  auto       Enable PAgP only if a PAgP device is detected
  desirable  Enable PAgP unconditionally
  on         Enable Etherchannel only
  passive    Enable LACP only if a LACP device is detected
ASW1(config-if-range)#channel-group 1 mode active
%EC-5-L3DONTBNDL2: Gig0/1 suspended: LACP currently not enabled on the remote port.
%EC-5-L3DONTBNDL2: Gig0/2 suspended: LACP currently not enabled on the remote port.
ASW1(config-if-range)#
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/2, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/2, changed state to up
```

1. I will now configure it as a trunk 

```jsx
ASW1(config-if)#sw mode trunk
ASW1(config-if)#
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/2, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/2, changed state to up
```

1. We will now check the etherchannel summary 

```jsx
ASW1(config-if)#do sh ether sum
Flags:  D - down        P - in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      f - failed to allocate aggregator
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port

Number of channel-groups in use: 1
Number of aggregators:           1

Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------

1      Po1(SD)           LACP   Gig0/1(I) Gig0/2(I) 
```

1. The flags shown:
    1. S: Switchport (Layer 2)
    2. D: Down
    3. I: Stand alone
2. It is currently down and each interface is acting as single ports, why?
    1.  We haven’t configured DSW1 yet

I didn’t write my notes too well and had to actually look at the way Jeremy did it *sigh* 😔 It’s okay, I’ll update my notes.

### DSW1 LACP Configuration

1. Choose the interfaces that will be in the EtherChannel and configure them to mode active for LACP

```jsx
DSW1>en
DSW1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
DSW1(config)#int range g1/0/3 - 4
DSW1(config-if-range)#channel-group 1 mode ?
  active     Enable LACP unconditionally
  auto       Enable PAgP only if a PAgP device is detected
  desirable  Enable PAgP unconditionally
  on         Enable Etherchannel only
  passive    Enable LACP only if a LACP device is detected
DSW1(config-if-range)#channel-group 1 mode active
%EC-5-L3DONTBNDL2: Gig1/0/3 suspended: LACP currently not enabled on the remote port.
%EC-5-L3DONTBNDL2: Gig1/0/4 suspended: LACP currently not enabled on the remote port.
DSW1(config-if-range)#
Creating a port-channel interface Port-channel 1

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0/3, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0/3, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0/4, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0/4, changed state to up

%LINK-5-CHANGED: Interface Port-channel1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel1, changed state to up
```

1. Still shows that the interfaces are down since switchport is not a trunk yet 

```jsx
DSW1(config-if-range)#int po1
DSW1(config-if)#sw mode trunk
DSW1(config-if)#
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0/3, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0/3, changed state to up

%LINK-3-UPDOWN: Interface Port-channel1, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel1, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0/4, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0/4, changed state to up

%LINK-5-CHANGED: Interface Port-channel1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel1, changed state to up
```

1. Spanning tree now shows that the interfaces are Po1 

```jsx
DSW1(config-if)#do sh spanni
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    20481
             Address     0007.EC07.1D30
             Cost        4
             Port        1(GigabitEthernet1/0/1)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    24577  (priority 24576 sys-id-ext 1)
             Address     0002.161B.EBBC
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi1/0/1          Root FWD 4         128.1    P2p
Gi1/0/2          Altn BLK 4         128.2    P2p
Po1              Desg FWD 3         128.29   P2p
```

1. And that it is now up and running, and interfaces are in the port-channel 

```jsx
DSW1(config-if)#do sh eth sum
Flags:  D - down        P - in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      f - failed to allocate aggregator
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port

Number of channel-groups in use: 1
Number of aggregators:           1

Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------

1      Po1(SU)           LACP   Gig1/0/3(P) Gig1/0/4(P) 
```

### ASW2 PAgP

1. I will now do the same to ASW2 without looking at my notes 
2. Buuuut one last thing I wanted to check was if I had to create another channel
    1. I can confirm that you do not 🙂‍↕️

```jsx
ASW2>en
ASW2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
ASW2(config)#int range g0/1 - 2
ASW2(config-if-range)#channel-group 1 mode ?
  active     Enable LACP unconditionally
  auto       Enable PAgP only if a PAgP device is detected
  desirable  Enable PAgP unconditionally
  on         Enable Etherchannel only
  passive    Enable LACP only if a LACP device is detected
ASW2(config-if-range)#channel-group 1 mode desirable
%EC-5-L3DONTBNDL2: Gig0/1 suspended: PAGP currently not enabled on the remote port.
%EC-5-L3DONTBNDL2: Gig0/2 suspended: PAGP currently not enabled on the remote port.
ASW2(config-if-range)#
Creating a port-channel interface Port-channel 1

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/2, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/2, changed state to up
```

1. Configured PAgP using ‘desirable’
2. Make it into a trunk 

```jsx
ASW2(config-if-range)#int po1
ASW2(config-if)#sw mode trunk
```

1. Verify that it was created 

```jsx
ASW2(config-if)#do sh eth sum
Flags:  D - down        P - in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      f - failed to allocate aggregator
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port

Number of channel-groups in use: 1
Number of aggregators:           1

Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------

1      Po1(SD)           PAgP   Gig0/1(I) Gig0/2(I) 
```

1. Yay! Created but down… we know what to do 😏

### DSW2 PAgP Config

1. Configure the right interfaces with PAgP in channel-group 1 using ‘desirable’ 

```jsx
DSW2>en
DSW2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
DSW2(config)#int range g1/0/3 - 4
DSW2(config-if-range)#channel-group 1 mode desirable
%EC-5-L3DONTBNDL2: Gig1/0/3 suspended: PAGP currently not enabled on the remote port.
%EC-5-L3DONTBNDL2: Gig1/0/4 suspended: PAGP currently not enabled on the remote port.
DSW2(config-if-range)#
Creating a port-channel interface Port-channel 1

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0/3, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0/3, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0/4, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0/4, changed state to up

%LINK-5-CHANGED: Interface Port-channel1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel1, changed state to up
```

1. Configure it as a trunk to make the ports active 

```jsx
DSW2(config-if-range)#int po1
DSW2(config-if)#sw mode trunk
DSW2(config-if)#
%LINK-3-UPDOWN: Interface Port-channel1, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel1, changed state to down

%LINK-5-CHANGED: Interface Port-channel1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel1, changed state to up
```

1. Confirm that the channel was created

```jsx
DSW2(config-if)#do sh sp
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    20481
             Address     0007.EC07.1D30
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    20481  (priority 20480 sys-id-ext 1)
             Address     0007.EC07.1D30
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Po1              Desg LIS 3         128.29   P2p
Gi1/0/2          Desg FWD 4         128.2    P2p
Gi1/0/1          Desg FWD 4         128.1    P2p
```

```jsx
DSW2(config-if)#do sh eth sum
Flags:  D - down        P - in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      f - failed to allocate aggregator
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port

Number of channel-groups in use: 1
Number of aggregators:           1

Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------

1      Po1(SU)           PAgP   Gig1/0/3(P) Gig1/0/4(P) 
```

## Configure Layer 3 EtherChannel Between DSW1 & DSW2

### DSW1 Static

1. Choose the range
2. disable switchport to make it into a layer 3 switch
3. Create a new group channel with mode ‘on’ for a static EtherChannel since the SW already has a channel-group 1 

```jsx
DSW1(config)#int range g1/0/1 - 2
DSW1(config-if-range)#no sw
DSW1(config-if-range)#channel-group 1 mode on
Command rejected (Port-channel): Either port is L2 and port-channel is L3, or vice-versa
Command rejected (Port-channel): Either port is L2 and port-channel is L3, or vice-versa
DSW1(config-if-range)#channel-group 2 mode on
DSW1(config-if-range)#
Creating a port-channel interface Port-channel 2

%LINK-5-CHANGED: Interface Port-channel2, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel2, changed state to up

```

1. Configure the ip address 

```jsx
DSW1>en
DSW1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
DSW1(config)#int range g1/0/1 - 2
DSW1(config-if-range)#no sw
DSW1(config-if-range)#channel-group 2 mode on
DSW1(config-if-range)#int po2
DSW1(config-if)#ip add 10.0.0.1 255.255.255.252
```

### DSW2 Static Config

1. Do the same for DSW2 

```jsx
DSW2>en
DSW2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
DSW2(config)#int range g1/0/1 - 2
DSW2(config-if-range)#no sw
Command rejected (Port-channel): Either port is L2 and port-channel is L3, or vice-versa
DSW2(config-if-range)#
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0/1, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel2, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel2, changed state to up

DSW2(config-if-range)#int po2
DSW2(config-if)#ip add 10.0.0.2 255.255.255.252
```

1. Verify that layer 3 is configured 

```jsx
DSW1(config-if)#do sh eth sum
Flags:  D - down        P - in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      f - failed to allocate aggregator
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port

Number of channel-groups in use: 2
Number of aggregators:           2

Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------

1      Po1(SU)           LACP   Gig1/0/3(P) Gig1/0/4(P) 
2      Po2(RU)           -      Gig1/0/1(P) Gig1/0/2(P) 
```

## Configure Routes for PCs to Reach SRV1

1. I will enable routing on both the multiswitches (Layer 3)

DSW1:

```jsx
DSW1(config)#ip routing
DSW1(config)#ip route 172.16.2.0 255.255.255.0 10.0.0.2
DSW1(config)#do sh ip rout
Codes: C - connected, S - static, I - IGRP, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is not set

     10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C       10.0.0.0/30 is directly connected, Port-channel2
L       10.0.0.1/32 is directly connected, Port-channel2
     172.16.0.0/16 is variably subnetted, 3 subnets, 2 masks
C       172.16.1.0/24 is directly connected, Vlan1
L       172.16.1.254/32 is directly connected, Vlan1
S       172.16.2.0/24 [1/0] via 10.0.0.2
```

DSW2:

```jsx
DSW2(config)#ip routing
DSW2(config)#do sh ip rou
Codes: C - connected, S - static, I - IGRP, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is not set

     10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C       10.0.0.0/30 is directly connected, Port-channel2
L       10.0.0.2/32 is directly connected, Port-channel2
     172.16.0.0/16 is variably subnetted, 2 subnets, 2 masks
C       172.16.2.0/24 is directly connected, Vlan1
L       172.16.2.254/32 is directly connected, Vlan1

DSW2(config)#ip route 172.16.1.0 255.255.255.0 10.0.0.1
DSW2(config)#do sh ip rout
Codes: C - connected, S - static, I - IGRP, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is not set

     10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C       10.0.0.0/30 is directly connected, Port-channel2
L       10.0.0.2/32 is directly connected, Port-channel2
     172.16.0.0/16 is variably subnetted, 3 subnets, 2 masks
S       172.16.1.0/24 [1/0] via 10.0.0.1
C       172.16.2.0/24 is directly connected, Vlan1
L       172.16.2.254/32 is directly connected, Vlan1
```

1. PC1 can now talk to Server

### What is the default EtherChannel load-balancing method used on each switch?

```jsx
ASW1#sh etherchannel load-balance
EtherChannel Load-Balancing Operational State (src-mac):
Non-IP: Source MAC address
  IPv4: Source MAC address
  IPv6: Source MAC address
```

1. This means that the switch uses src-mac to load balance

### Configure the switches to load-balance based on source and destination IP addresses.

ASW1:

```jsx
ASW1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
ASW1(config)#port-channel load
ASW1(config)#port-channel load-balance ?
  dst-ip       Dst IP Addr
  dst-mac      Dst Mac Addr
  src-dst-ip   Src XOR Dst IP Addr
  src-dst-mac  Src XOR Dst Mac Addr
  src-ip       Src IP Addr
  src-mac      Src Mac Addr
ASW1(config)#port-channel load-balance src-dst-ip
ASW1(config)#do sh ether load
EtherChannel Load-Balancing Operational State (src-dst-ip):
Non-IP: Source XOR Destination MAC address
  IPv4: Source XOR Destination IP address
  IPv6: Source XOR Destination IP address
```

ASW2:

```jsx
ASW2>en
ASW2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
ASW2(config)#port-ch
ASW2(config)#port-channel load
ASW2(config)#port-channel load-balance src-dst-ip
ASW2(config)#do sh etherc
ASW2(config)#do sh etherc LOAD
EtherChannel Load-Balancing Operational State (src-dst-ip):
Non-IP: Source XOR Destination MAC address
  IPv4: Source XOR Destination IP address
  IPv6: Source XOR Destination IP address
```