# Day 24 - Floating Static Routes

![image.png](image.png)

## Check the routing tables of R1 and R2

1. Which dynamic routing protocol is Enterprise A using? 
    1. Enterprise A is using OSPF as shown with the code ‘O’ and AD of 110

```jsx
R1#sh ip rout
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is 203.0.113.9 to network 0.0.0.0

     10.0.0.0/8 is variably subnetted, 5 subnets, 3 masks
C       10.0.0.0/30 is directly connected, GigabitEthernet0/2/0
L       10.0.0.1/32 is directly connected, GigabitEthernet0/2/0
C       10.0.1.0/24 is directly connected, GigabitEthernet0/1
L       10.0.1.254/32 is directly connected, GigabitEthernet0/1
O       10.0.2.0/24 [110/2] via 10.0.0.2, 00:14:13, GigabitEthernet0/2/0
     203.0.113.0/24 is variably subnetted, 4 subnets, 2 masks
C       203.0.113.0/30 is directly connected, GigabitEthernet0/0/0
L       203.0.113.2/32 is directly connected, GigabitEthernet0/0/0
C       203.0.113.8/30 is directly connected, GigabitEthernet0/1/0
L       203.0.113.10/32 is directly connected, GigabitEthernet0/1/0
S*   0.0.0.0/0 [1/0] via 203.0.113.9
```

```jsx
R2#sh ip route 
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is 203.0.113.13 to network 0.0.0.0

     10.0.0.0/8 is variably subnetted, 5 subnets, 3 masks
C       10.0.0.0/30 is directly connected, GigabitEthernet0/2/0
L       10.0.0.2/32 is directly connected, GigabitEthernet0/2/0
O       10.0.1.0/24 [110/2] via 10.0.0.1, 00:18:15, GigabitEthernet0/2/0
C       10.0.2.0/24 is directly connected, GigabitEthernet0/1
L       10.0.2.254/32 is directly connected, GigabitEthernet0/1
     203.0.113.0/24 is variably subnetted, 4 subnets, 2 masks
C       203.0.113.4/30 is directly connected, GigabitEthernet0/0/0
L       203.0.113.6/32 is directly connected, GigabitEthernet0/0/0
C       203.0.113.12/30 is directly connected, GigabitEthernet0/1/0
L       203.0.113.14/32 is directly connected, GigabitEthernet0/1/0
S*   0.0.0.0/0 [1/0] via 203.0.113.13
```

1. Which route will be used if PC1 tries to access SRV1? 
    1. The routing table’s best route will be through R1. The next hop will be to R2 via 10.0.0.2 through GigabitEthernet0/2/0

```jsx
O       10.0.2.0/24 [110/2] via 10.0.0.2, 00:14:13, GigabitEthernet0/2/0
```

1. Which route will be used if PC1 tries to access remote server 1.1.1.1 over the Internet?
    1. Closest route to the internet would be through ISPBR1 through the default gateway 
    
    ```jsx
    S*   0.0.0.0/0 [1/0] via 203.0.113.9
    ```
    
2. Test by pinging SRV1 and 1.1.1.1 

## Configure floating static routes on R1 and R2 that

1. allow PC1 to reach SRV1 if the link between R1 and R2 fails.
    1. Need to first configure an ip route that has a higher metric than R2 
    2. Since the R2 metric is 110, we set the administrative distance to SPR1 to 111
    
    ```jsx
    R1#conf t
    Enter configuration commands, one per line.  End with CNTL/Z.
    R1(config)#ip route 10.0.2.0 255.255.255.0 203.0.113.1 ?
      <1-255>  Distance metric for this route
      <cr>
    R1(config)#ip route 10.0.2.0 255.255.255.0 203.0.113.1 111
    ```
    
    c. Then we do the same to R2 and update the path incase the connection between R1 and R2 fail. 
    
    ```jsx
    R2(config)#ip route 10.0.1.0 255.255.255.0 203.0.113.4 111
    ```
    
2. Do the routes enter the routing tables of R1 and R2?
    1. Before shutting off the interface I still saw g0/2/0 interfaces using the original route 
    
    ```jsx
    R1(config)#do sh ip route
    Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
           D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
           N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
           E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
           i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
           * - candidate default, U - per-user static route, o - ODR
           P - periodic downloaded static route
    
    Gateway of last resort is 203.0.113.9 to network 0.0.0.0
    
         10.0.0.0/8 is variably subnetted, 5 subnets, 3 masks
    C       10.0.0.0/30 is directly connected, GigabitEthernet0/2/0
    L       10.0.0.1/32 is directly connected, GigabitEthernet0/2/0
    C       10.0.1.0/24 is directly connected, GigabitEthernet0/1
    L       10.0.1.254/32 is directly connected, GigabitEthernet0/1
    O       10.0.2.0/24 [110/2] via 10.0.0.2, 00:40:33, GigabitEthernet0/2/0
    ```
    

### Shut down the G0/2/0 interface of R1 or R2.

1. I then shutdown the interface and saw that the new static route I configured is now active 

```jsx
R1#sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is 203.0.113.9 to network 0.0.0.0

     10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
C       10.0.1.0/24 is directly connected, GigabitEthernet0/1
L       10.0.1.254/32 is directly connected, GigabitEthernet0/1
S       10.0.2.0/24 [111/0] via 203.0.113.1
```

1. Do the floating static routes enter the routing tables of R1 and R2?
    1. As I stated above the static route made it into the table after I shutdown the interface connecting R1 and R2
2. Ping from PC1 to SRV1 to confirm.
    1. When I did this, all my traffic got lost 😭
    2. I then used the command ‘tracert’ to see where my traffic was losing the packets.
    3. I realized that my traffic stopped at SPR2 😕 
    
    ```jsx
    C:\>tracert 10.0.2.1
    
    Tracing route to 10.0.2.1 over a maximum of 30 hops: 
    
      1   0 ms      0 ms      0 ms      10.0.1.254
      2   0 ms      0 ms      0 ms      203.0.113.1
      3   0 ms      0 ms      0 ms      192.168.1.2
      4   *         *         *         Request timed out.
      5   *         *         *         Request timed out.
      6   *         *         *         Request timed out.
      7   *         *         *         Request timed out.
    ```
    
    1. I looked back on how I configured the route for R2 and realized I made a tiny mistake 🥲 
        1. I originally set the next hop to
            
            ```jsx
            R2(config)#ip route 10.0.1.0 255.255.255.0 203.0.113.4 111
            ```
            
        2. When it should be 
            
            ```jsx
            R2(config)#ip route 10.0.1.0 255.255.255.0 203.0.113.5 111
            ```
            
    
    2. It works now 🙂‍↕️
    
    ```jsx
    C:\>tracert 10.0.2.1
    
    Tracing route to 10.0.2.1 over a maximum of 30 hops: 
    
      1   0 ms      0 ms      0 ms      10.0.1.254
      2   0 ms      0 ms      0 ms      203.0.113.1
      3   0 ms      0 ms      0 ms      192.168.1.2
      4   0 ms      0 ms      0 ms      203.0.113.6
      5   0 ms      0 ms      0 ms      10.0.2.1
    
    Trace complete.
    ```