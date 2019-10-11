# Mikrotik MPLS L2VPN and L3VPN

## Topology
![topology.png](https://raw.githubusercontent.com/udienz/gns3-lab/master/mikrotik-mpls/topology.png)

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

## Test

### MPLS at Provider

```
[mahyuddin@R1] > mpls ldp neighbor pr
Flags: X - disabled, D - dynamic, O - operational, T - sending-targeted-hello, 
V - vpls 
 #      TRANSPORT       LOCAL-TRANSPORT PEER                       SEN
 0 DOTV 100.121.0.4     100.121.0.1     100.121.0.4:0              yes
 1 DOTV 100.121.0.2     100.121.0.1     100.121.0.2:0              yes
 2 DOTV 100.121.0.3     100.121.0.1     100.121.0.3:0              yes
[mahyuddin@R1] > interface traffic-eng print 
Flags: X - disabled, D - dynamic, R - running 
 0  R name="te-r1-r2" mtu=1500 disable-running-check=no 
      from-address=100.121.0.1 to-address=100.121.0.2 bandwidth=100Mbps 
      primary-path=te-tp-dyn secondary-paths="" primary-retry-interval=1m 
      bandwidth-limit=disabled auto-bandwidth-range=0bps 
      auto-bandwidth-reserve=0% auto-bandwidth-avg-interval=5m 
      auto-bandwidth-update-interval=1h 

 1  R name="te-r1-r3" mtu=1500 disable-running-check=no 
      from-address=100.121.0.1 to-address=100.121.0.3 bandwidth=100Mbps 
      primary-path=te-tp-dyn secondary-paths="" primary-retry-interval=1m 
      bandwidth-limit=disabled auto-bandwidth-range=0bps 
      auto-bandwidth-reserve=0% auto-bandwidth-avg-interval=5m 
      auto-bandwidth-update-interval=1h 

 2  R name="te-r1-r4" mtu=1500 disable-running-check=no 
      from-address=100.121.0.1 to-address=100.121.0.4 bandwidth=100Mbps 
      primary-path=te-tp-dyn secondary-paths="" primary-retry-interval=1m 
      bandwidth-limit=disabled auto-bandwidth-range=0bps 
      auto-bandwidth-reserve=0% auto-bandwidth-avg-interval=5m 
      auto-bandwidth-update-interval=1h 
[mahyuddin@R1] > interface vpls print
Flags: X - disabled, R - running, D - dynamic, 
B - bgp-signaled, C - cisco-bgp-signaled 
 0 R   name="vpls-e6-r2" mtu=1500 l2mtu=1500 mac-address=02:F4:CB:23:BC:E9 
       arp=enabled arp-timeout=auto disable-running-check=no 
       remote-peer=100.121.0.2 vpls-id=6:12 cisco-style=no cisco-style-id=0 
       advertised-l2mtu=1500 pw-type=raw-ethernet use-control-word=default 

 1 R   name="vpls-e6-r3" mtu=1500 l2mtu=1500 mac-address=02:F4:CB:23:BC:E9 
       arp=enabled arp-timeout=auto disable-running-check=no 
       remote-peer=100.121.0.3 vpls-id=6:13 cisco-style=no cisco-style-id=0 
       advertised-l2mtu=1500 pw-type=raw-ethernet use-control-word=default 

 2 R   name="vpls-e6-r4" mtu=1500 l2mtu=1500 mac-address=02:F4:CB:23:BC:E9 
       arp=enabled arp-timeout=auto disable-running-check=no 
       remote-peer=100.121.0.4 vpls-id=6:14 cisco-style=no cisco-style-id=0 
       advertised-l2mtu=1500 pw-type=raw-ethernet use-control-word=default 
[mahyuddin@R1] > interface vpls monitor 0 once 
       remote-label: 23
        local-label: 22
      remote-status: 
          transport: te-r1-r2
  transport-nexthop: 100.121.12.2
     imposed-labels: 0,23

```
### L2VPN

```
[mahyuddin@cust-vpls-a-r3] > /ip neighbor print value-list
              interface: ether6                ether6                ether6                ether6                                ether6            ether6            ether6
                address:                                                                                                         100.122.6.1       100.122.6.2       100.122.6.4
               address4:                                                                                                         100.122.6.1       100.122.6.2       100.122.6.4
            mac-address: 02:33:8B:BD:EB:4C     02:65:38:35:42:69     02:F4:CB:23:BC:E9     0C:06:63:11:91:05                     0C:06:63:91:9D:05 0C:06:63:DD:AA:05 0C:06:63:A6:E7:05
               identity: R2                    R4                    R1                    R3                                    cust-vpls-a-r1    cust-vpls-a-r2    cust-vpls-a-r4
               platform: MikroTik              MikroTik              MikroTik              MikroTik                              MikroTik          MikroTik          MikroTik
                version: 6.44.2 (stable)       6.44.2 (stable)       6.44.2 (stable)       6.44.2 (stable)                       6.44.2 (stable)   6.44.2 (stable)   6.44.2 (stable)
                 unpack: none                  none                  none                  none                                  none              none              none
                    age: 59s                   53s                   55s                   53s                                   58s               53s               53s
                 uptime: 38m13s                38m14s                38m14s                38m14s                                38m13s            38m14s            38m14s
            software-id: yd/KupPs6aL           HccicVMUcOM           dOqXkiDOyGB           MZM6StnwBCD                           41G6vK3FMGL       IUpxQzQQaXF       nuRCrw0c6iO
                  board: CHR                   CHR                   CHR                   CHR                                   CHR               CHR               CHR
                   ipv6: no                    no                    no                                                          no                no                no
         interface-name: br-vpls-e6/vpls-e6-r1 br-vpls-e6/vpls-e6-r1 br-vpls-e6/vpls-e6-r3 br-vpls-e6/ether6                     ether6            ether6            ether6
     system-description:                                                                   MikroTik RouterOS 6.44.2 (stable) CHR
            system-caps:                                                                   bridge                                                                    
                                                                                           router                                                                    
    system-caps-enabled:                                                                   bridge
                                                                                           router                                                                    
[mahyuddin@cust-vpls-a-r3] > ping 100.122.6.2
  SEQ HOST                                     SIZE TTL TIME  STATUS
    0 100.122.6.2                                56  64 21ms 
    1 100.122.6.2                                56  64 11ms 
    sent=2 received=2 packet-loss=0% min-rtt=11ms avg-rtt=16ms max-rtt=21ms 

[mahyuddin@cust-vpls-a-r3] > 
```

### L3VPN

```
[mahyuddin@cust-vrf-b-r3] > /ip neighbor print value-list 
            interface: ether7
              address: 100.123.73.13
             address4: 100.123.73.13
          mac-address: 0C:06:63:11:91:06
             identity: R3
             platform: MikroTik
              version: 6.44.2 (stable)
               unpack: none
                  age: 13s
               uptime: 41m14s
          software-id: MZM6StnwBCD
                board: CHR
                 ipv6: no
       interface-name: ether7
   system-description: MikroTik RouterOS 6.44.2 (stable) CHR
          system-caps: bridge,router
  system-caps-enabled: bridge,router

[mahyuddin@cust-vrf-b-r3] > ping 100.123.0.1
  SEQ HOST                                     SIZE TTL TIME  STATUS
    0 100.123.0.1                                56  62 9ms
    1 100.123.0.1                                56  62 7ms
    sent=2 received=2 packet-loss=0% min-rtt=7ms avg-rtt=8ms max-rtt=9ms 

[mahyuddin@cust-vrf-b-r3] > /tool traceroute 1.1.1.1
 # ADDRESS                          LOSS SENT    LAST     AVG    BEST   WORST
 1 100.120.23.3                       0%    4   9.7ms     9.8     9.1    10.5
 2                                  100%    4 timeout
 3 100.123.71.1                       0%    3     9ms       9     8.4     9.7
 4 100.123.11.14                      0%    3  10.6ms    10.2     9.8    10.6
 5 100.121.9.1                        0%    3  13.1ms    13.1      13    13.1
 6                                    0%    0     0ms

```
