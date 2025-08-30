# Routing-IPv4-with-BGP

in this lab we are going to connect enterprise network out to a couple of different service providers using BGP

goal is Connect R3 in the enterprise network (AS 65001) to R2, ISP1, and ISP2 using BGP

<img width="1562" height="672" alt="image" src="https://github.com/user-attachments/assets/cfaf3bc5-7bd9-46ea-af2a-8a85afb984c7" />


BGP Configuration Project – Router R3
Project Overview

Earlier in this module, we looked at a basic BGP configuration for IPv4. In this project, I expanded the lab to a more complex topology where most of the routers are already configured with BGP, except Router R3.

The goal of this exercise was to:

Configure R3 to advertise its internal network.

Form BGP neighbor relationships with R2, ISP1, and ISP2.

Prevent R3 from becoming a transit router between the ISPs.

Verify BGP operation, troubleshoot routing issues, and apply optimizations such as next-hop-self, route reflectors, and peer groups.

Step 1 – Basic BGP Setup on R3

On Router R3, I entered global configuration mode and started the BGP process:

router bgp 65001
 bgp router-id 3.3.3.3


The router ID is not tied to an interface; it’s simply a unique identifier (similar to OSPF).

Step 2 – Define Neighbors

I configured R3 to peer with three routers:

R2 (iBGP peer)

neighbor 10.1.1.2 remote-as 65001


ISP1 (eBGP peer)

neighbor 198.51.100.2 remote-as 65002


ISP2 (eBGP peer)

neighbor 198.51.100.6 remote-as 65004


This successfully established BGP adjacencies with R2, ISP1, and ISP2.

Step 3 – Advertise Internal Network

Only the 10.1.1.0/24 network (link between R2 and R3) should be advertised.

network 10.1.1.0 mask 255.255.255.0


This avoids making R3 a transit path between ISPs.

Step 4 – Verification

Checked routes with show ip bgp

Verified Internet reachability (represented by loopback 5.5.5.0/24 on the INET router).

Initially, R3 preferred the slower 10 Mbps ISP1 link instead of the faster 100 Mbps ISP2 link (BGP does not consider bandwidth by default).

Step 5 – Troubleshooting Next-Hop Issues

R2 learned about 5.5.5.0/24, but pings to 5.5.5.5 failed.

Root cause: R2 did not know how to reach the next hop (198.51.100.2).

Fix: On R3, I used next-hop-self when advertising routes to R2:

neighbor 10.1.1.2 next-hop-self


After clearing BGP (clear ip bgp *), R2 correctly used R3 (10.1.1.1) as the next hop and could reach 5.5.5.5.

Step 6 – Route Reflector

R1 could not reach external networks because iBGP by default does not re-advertise routes learned from another iBGP peer.

Solution: Configure R2 as a route reflector with R1 and R3 as clients:

router bgp 65001
 neighbor 172.16.1.1 route-reflector-client   ! R1
 neighbor 10.1.1.1 route-reflector-client    ! R3


Now R1 also learned routes to the Internet.

Step 7 – Peer Groups

To simplify configuration and reduce CPU load, I created a peer group for ISP neighbors. A prefix-list was applied to filter RFC1918 private addresses.

neighbor BGP-PG peer-group
neighbor BGP-PG prefix-list BGP-DEMO in
neighbor 198.51.100.2 peer-group BGP-PG
neighbor 198.51.100.6 peer-group BGP-PG


This ensured both ISPs shared the same inbound filtering policy.

Later, I removed the peer group since the next part of the lab required treating ISPs differently.

Final Verification

All routers in AS 65001 (R1, R2, R3) had reachability to 5.5.5.5.

R3 successfully established iBGP and eBGP peerings.

Routing was corrected with next-hop-self and route reflector configuration.

Peer groups were demonstrated for scalability and policy consistency.

Next Steps

The next phase of the project will focus on path selection optimization, ensuring traffic prefers the higher-bandwidth ISP2 link instead of ISP1.

✅ Result: Successfully configured and verified BGP on Router R3, established neighbor relationships, and implemented fixes for next-hop and route reflection issues.




