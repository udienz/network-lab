# Mikrotik MPLS L2VPN and L3VPN

## Topology

## Basic

We will setup provide L2VPN and L3VPN with Mikrotik, we also provide IP transit.
We have four datacenter and already support Jumbo frame, on each datacenter we will
provide L2/L3VPN

To make easy when imagine the datacenter, here is how i imagined:
1. First Datacenter, Located in Jakarta, all routers (including customer routers will use R1 in their prefix)
2. Second Datacenter, Located in Bandung, all routers will use R2 prefix
3. Third Datacenter, Located in Surabaya, all routers will use R3 prefix
4. Fourth Datacenter, located in Malang, all routers will use R4 prefix
5. Each customer will have dedicated router and connected to R1

## Goal
1. L2VPN: Customer need Layer 2 connection in each datacenter, all routers should connected
   R1 in Jakarta DC
2. L3VPN: Customer need layer 3 coonection in each datacenter, customer will connected to
   provider router with L3 with separate routing table as well as OSPF instance
