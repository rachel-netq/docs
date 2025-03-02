---
title: Equal Cost Multipath Load Sharing - Hardware ECMP
author: NVIDIA
weight: 770
toc: 3
---
Cumulus Linux enables hardware [ECMP](## "Equal Cost Multi Path") by default. Load sharing occurs automatically for all routes with multiple next hops installed. ECMP load sharing supports both IPv4 and IPv6 routes.

## How Does ECMP Work?

ECMP operates only on equal cost routes in the Linux routing table. In the following example, the 10.1.1.0/24 route has two possible next hops installed in the routing table:

```
cumulus@switch:~$ ip route show 10.1.1.0/24
10.1.1.0/24  proto zebra  metric 20
  nexthop via 192.168.1.1 dev swp1 weight 1 onlink
  nexthop via 192.168.2.1 dev swp2 weight 1 onlink
```

For Cumulus Linux to consider routes equal, they must:

- Originate from the same routing protocol. Routes from different sources are not considered equal. For example, a static route and an OSPF route are not considered for ECMP load sharing.
- Have equal cost. If two routes from the same protocol are unequal, only the best route installs in the routing table.

{{%notice note%}}
Cumulus Linux enables the BGP `maximum-paths` setting by default and installs multiple routes. Refer to {{<link url="Optional-BGP-Configuration#ecmp" text="BGP and ECMP">}}.
{{%/notice%}}

When multiple routes are in the routing table, a hash determines through which path a packet follows. Cumulus Linux hashes on the following fields:
- IP protocol
- Ingress interface
- Source IPv4 or IPv6 address
- Destination IPv4 or IPv6 address
- Source or destination MAC address
- Ethertype
- VLAN ID
- [TEID](## "Tunnel Endpoint Identifier")

For TCP and UDP frames, Cumulus Linux also hashes on the source port or destination port.

{{< img src = "/images/cumulus-linux/ecmp-packet-hash.png" >}}

To prevent out of order packets, ECMP hashes on a per-flow basis; all packets with the same source and destination IP addresses and the same source and destination ports always hash to the same next hop. ECMP hashing does not keep a record of flow states.

ECMP hashing does not keep a record of packets that hash to each next hop and does not guarantee that traffic to each next hop is equal.
<!-- vale off -->

## Custom Hashing

You can configure custom hashing to specify what to include in the hash calculation during ECMP load balancing between:
- Multiple next hops of a layer 3 route.
- Multiple interfaces that are members of the same bond.

For ECMP load balancing between multiple next-hops of a layer 3 route, you can hash on these fields:

|  Field  | NVUE Command | `/etc/cumulus/datapath/traffic.conf` Parameter|
| -------- | ----------- | --------------------------------------------- |
| IP protocol | `nv set system forwarding ecmp-hash ip-protocol`|`hash_config.ip_prot`|
| Source IP address| `nv set system forwarding ecmp-hash source-ip`|`hash_config.sip`|
| Destination IP address| `nv set system forwarding ecmp-hash destination-ip`|`hash_config.dip`|
| Source port | `nv set system forwarding ecmp-hash source-port`|`hash_config.sport` |
| Destination port| `nv set system forwarding ecmp-hash destination-port`| `hash_config.dport` |
| IPv6 flow label | `nv set system forwarding ecmp-hash ipv6-label`|`hash_config.ip6_label` |
| Ingress interface | `nv set system forwarding ecmp-hash ingress-interface`| `hash_config.ing_intf` |
| TEID (see {{<link url="#gtp-hashing" text="GTP Hashing, below" >}}) | `nv set system forwarding ecmp-hash gtp-teid`| `hash_config.gtp_teid`|
| Inner IP protocol| `nv set system forwarding ecmp-hash inner-ip-protocol `|`hash_config.inner_ip_prot` |
| Inner source IP address| `nv set system forwarding ecmp-hash inner-source-ip`|`hash_config.inner_sip` |
| Inner destination IP address| `nv set system forwarding ecmp-hash inner-destination-ip`|`hash_config.inner_dip` |
| Inner source port| `nv set system forwarding ecmp-hash inner-source-port`| `hash_config.inner-sport` |
| Inner destination port| `nv set system forwarding ecmp-hash inner-destination-port`| `hash_config.inner_dport` |
| Inner IPv6 flow label | `nv set system forwarding ecmp-hash inner-ipv6-label`|`hash_config.inner_ip6_label` |

The following example commands omit the source port and destination port from the hash calculation:

{{< tabs "TabID173 ">}}
{{< tab "NVUE Commands">}}

```
cumulus@switch:~$ nv set system forwarding ecmp-hash source-port
cumulus@switch:~$ nv set system forwarding ecmp-hash destination-port
cumulus@switch:~$ nv config apply
```

{{< /tab >}}
{{< tab "Linux Commands ">}}

1. Edit the `/etc/cumulus/datapath/traffic.conf` file:
   - Uncomment the `hash_config.enable = true` option.
   - Set the `hash_config.sport` and `hash_config.dport` options to `false`.

```
cumulus@switch:~$ sudo nano /etc/cumulus/datapath/traffic.conf
...
# HASH config for  ECMP to enable custom fields
# Fields will be applicable for ECMP hash
# calculation
#Note : Currently supported only for MLX platform
# Uncomment to enable custom fields configured below
hash_config.enable = true

#hash Fields available ( assign true to enable)
#ip protocol
hash_config.ip_prot = true
#source ip
hash_config.sip = true
#destination ip
hash_config.dip = true
#source port
hash_config.sport = false
#destination port
hash_config.dport = false
...
```

2. Run the `echo 1 > /cumulus/switchd/ctrl/hash_config_reload` command. This command does not cause any traffic interruptions.

   ```
   cumulus@switch:~$ echo 1 > /cumulus/switchd/ctrl/hash_config_reload
   ```

{{< /tab >}}
{{< /tabs >}}

For ECMP load balancing between multiple interfaces that are members of the same bond, you can hash on these fields:

|  Field  | NVUE Command | `/etc/cumulus/datapath/traffic.conf` Parameter|
| -------- | ----------- | --------------------------------------------- |
| IP protocol | `nv set system forwarding lag-hash ip-protocol`|`lag_hash_config.ip_prot`|
| Source MAC address| `nv set system forwarding lag-hash source-mac`|`lag_hash_config.smac`|
| Destination MAC address| `nv set system forwarding lag-hash destination-mac`|`lag_hash_config.dmac`|
| Source IP address | `nv set system forwarding lag-hash source-ip`|`lag_hash_config.sip` |
| Destination IP address| `nv set system forwarding lag-hash destination-ip`| `lag_hash_config.dip` |
| Source port | `nv set system forwarding lag-hash source-port`|`lag_hash_config.sport` |
| Destination port | `nv set system forwarding lag-hash destination-port`| `lag_hash_config.dport` |
| Ethertype| `nv set system forwarding lag-hash ether-type`|`lag_hash_config.ether_type` |
| VLAN ID| `nv set system forwarding lag-hash vlan`|`lag_hash_config.vlan_id` |
| TEID (see {{<link url="#gtp-hashing" text="GTP Hashing, below" >}}) | `nv set system forwarding lag-hash gtp-teid`| `lag_hash_config.gtp_teid`|

The following example commands omit the source MAC address and destination MAC address from the hash calculation:

{{< tabs "TabID149 ">}}
{{< tab "NVUE Commands">}}

```
cumulus@switch:~$ nv set system forwarding lag-hash source-mac
cumulus@switch:~$ nv set system forwarding lag-hash destination-mac
cumulus@switch:~$ nv config apply
```

{{< /tab >}}
{{< tab "Linux Commands ">}}

1. Edit the `/etc/cumulus/datapath/traffic.conf` file:
   - Uncomment the `lag_hash_config.enable` option.
   - Set the `lag_hash_config.smac` and `lag_hash_config.dmac` options to `false`.

```
cumulus@switch:~$ sudo nano /etc/cumulus/datapath/traffic.conf
...
#LAG HASH config
#HASH config for LACP to enable custom fields
#Fields will be applicable for LAG hash
#calculation
#Uncomment to enable custom fields configured below
#lag_hash_config.enable = true

lag_hash_config.smac = false
lag_hash_config.dmac = false
lag_hash_config.sip  = true
lag_hash_config.dip  = true
lag_hash_config.ether_type = true
lag_hash_config.vlan_id = true
lag_hash_config.sport = true
lag_hash_config.dport = true
lag_hash_config.ip_prot = true
#GTP-U teid
lag_hash_config.gtp_teid = false
...
```

2. Run the `echo 1 > /cumulus/switchd/ctrl/hash_config_reload` command. This command does not cause any traffic interruptions.

   ```
   cumulus@switch:~$ echo 1 > /cumulus/switchd/ctrl/hash_config_reload
   ```

{{< /tab >}}
{{< /tabs >}}

<!-- vale off -->

{{%notice note%}}
Cumulus Linux enables symmetric hashing by default. Make sure that the settings for the source IP and destination IP fields match, and that the settings for the source port and destination port fields match; otherwise Cumulus Linux disables symmetric hashing automatically. If necessary, you can disable symmetric hashing manually in the `/etc/cumulus/datapath/traffic.conf` file by setting `symmetric_hash_enable = FALSE`.
{{%/notice%}}

## GTP Hashing
<!-- vale on -->
[GTP](## "GPRS Tunneling Protocol") carries mobile data within the core of the mobile operator’s network. Traffic in the 5G Mobility core cluster, from cell sites to compute nodes, have the same source and destination IP address. The only way to identify individual flows is with the GTP [TEID](## "Tunnel Endpoint Identifier"). Enabling GTP hashing adds the TEID as a hash parameter and helps the Cumulus Linux switches in the network to distribute mobile data traffic evenly across ECMP routes.

Cumulus Linux supports TEID-based *ECMP hashing* for:
- [GTP-U](## "GPRS Tunnelling Protocol User") packets ingressing physical ports or bonds.
- VXLAN encapsulated GTP-U packets terminating on egress [VTEPs](## "Virtual Tunnel End Points").

Cumulus Linux supports TEID-based *load balancing* for traffic egressing a bond.

GTP TEID-based ECMP hashing and load balancing is only applicable if the outer header egressing the port is GTP encapsulated and if the ingress packet is either a GTP-U packet or a VXLAN encapsulated GTP-U packet.

{{%notice note%}}
- Cumulus Linux supports GTP Hashing on NVIDIA Spectrum-2 and later.
- [GTP-C](## "GPRS Tunnelling Protocol Control") packets are not part of GTP hashing.
{{%/notice%}}

To enable TEID-based ECMP hashing:

{{< tabs "TabID221 ">}}
{{< tab "NVUE Commands">}}

```
cumulus@switch:~$ nv set system forwarding ecmp-hash gtp-teid
cumulus@switch:~$ nv config apply
```

To disable TEID-based ECMP hashing, run the `nv set system forwarding ecmp-hash gtp-teid` command.

{{< /tab >}}
{{< tab "Linux Commands ">}}

1. Edit the `/etc/cumulus/datapath/traffic.conf` file and change the `lag_hash_config.gtp_teid` parameter to `true`:

   ```
   cumulus@switch:~$ sudo nano /etc/cumulus/datapath/traffic.conf
   ...
   #GTP-U teid
   hash_config.gtp_teid = true
   ```

2. Run the `echo 1 > /cumulus/switchd/ctrl/hash_config_reload` command. This command does not cause any traffic interruptions.

   ```
   cumulus@switch:~$ echo 1 > /cumulus/switchd/ctrl/hash_config_reload
   ```

To disable TEID-based ECMP hashing, set the `hash_config.gtp_teid` parameter to `false`, then reload the configuration.

{{< /tab >}}
{{< /tabs >}}

To enable TEID-based load balancing:

{{< tabs "TabID256 ">}}
{{< tab "NVUE Commands">}}

```
cumulus@switch:~$ nv set system forwarding lag-hash gtp-teid
cumulus@switch:~$ nv config apply
```

To disable TEID-based load balancing, run the `nv set system forwarding lag-hash gtp-teid` command.

{{< /tab >}}
{{< tab "Linux Commands ">}}

1. Edit the `/etc/cumulus/datapath/traffic.conf` file:
   - Uncomment the `hash_config.enable = true` line.
   - Change the `lag_hash_config.gtp_teid` parameter to `true`.

   ```
   cumulus@switch:~$ sudo nano /etc/cumulus/datapath/traffic.conf
   ...
   # Uncomment to enable custom fields configured below
   hash_config.enable = true
   ...
   #GTP-U teid
   lag_hash_config.gtp_teid = true
   ```

2. Run the `echo 1 > /cumulus/switchd/ctrl/hash_config_reload` command. This command does not cause any traffic interruptions.

   ```
   cumulus@switch:~$ echo 1 > /cumulus/switchd/ctrl/hash_config_reload
   ```

To disable TEID-based load balancing, set the `lag_hash_config.gtp_teid` parameter to `false`, then reload the configuration.

{{< /tab >}}
{{< /tabs >}}

## Unique Hash Seed

You can configure a unique hash seed for each switch to prevent *hash polarization*, a type of network congestion that occurs when multiple data flows try to reach a switch using the same switch ports.

You can set a hash seed value between 0 and 4294967295. If you do not specify a value, `switchd` creates a randomly generated seed.

The following example commands configure the hash seed to 50.

{{< tabs "TabID125 ">}}
{{< tab "NVUE Commands">}}

```
cumulus@switch:~$ nv set system forwarding hash-seed 50
cumulus@switch:~$ nv config apply
```

{{< /tab >}}
{{< tab "Linux Commands ">}}

Edit `/etc/cumulus/datapath/traffic.conf` file to change the `ecmp_hash_seed` parameter, then restart `switchd`.

```
cumulus@switch:~$ sudo nano /etc/cumulus/datapath/traffic.conf
...
#Specify the hash seed for Equal cost multipath entries
# and for custom ecmp and lag hash
# Default value: random
# Value Range: {0..4294967295}
ecmp_hash_seed = 50
...
```
<!-- vale off -->
{{<cl/restart-switchd>}}
<!-- vale on -->

{{< /tab >}}
{{< /tabs >}}

## Resilient Hashing

In Cumulus Linux, when a next hop fails or you remove the next hop from an ECMP pool, the hashing or hash bucket assignment can change. *Resilient hashing* is an alternate way to manage ECMP groups. Cumulus Linux assigns next hops to buckets using their hashing header fields and uses the resulting hash to index into the table of 2^n hash buckets. Because all packets in a given flow have the same header hash value, they all use the same flow bucket.

{{%notice note%}}
- Resilient hashing supports both IPv4 and IPv6 routes.
- Resilient hashing prevents disruption when you remove next hops but does not prevent disruption when you add next hops.
{{%/notice%}}

The NVIDIA Spectrum ASIC assigns packets to hash buckets and assigns hash buckets to next hops. The ASIC also runs a background thread that monitors buckets and can migrate buckets between next hops to rebalance the load.
- When you remove a next hop, Cumulus Linux distributes the assigned buckets to the remaining next hops.
- When you add a next hop, Cumulus Linux assigns **no** buckets to the new next hop until the background thread rebalances the load.
- The load rebalances when the active flow timer expires only if there are inactive hash buckets available; the new next hop can remain unpopulated until the period set in active flow timer expires.
- When the unbalanced timer expires and the load is not balanced, the thread migrates buckets to different next hops to rebalance the load.

Any flow can migrate to any next hop, depending on flow activity and load balance conditions. Over time, the flow can get pinned, which is the default setting and behavior.

When you enable resilient hashing, Cumulus Linux assigns next hops in round robin fashion to a fixed number of buckets. In this example, there are 12 buckets and four next hops.

{{< img src = "/images/cumulus-linux/ecmp-reshash-bucket-assignment.png" >}}

Unlike default ECMP hashing, when you need to remove a next hop, the number of hash buckets does not change.

{{< img src = "/images/cumulus-linux/ecmp-reshash-failure.png" >}}

With 12 buckets and four next hops, instead of reducing the number of buckets, which impacts flows to known good hosts, the remaining next hops replace the failed next hop.

{{< img src = "/images/cumulus-linux/ecmp-reshash-restore.png" >}}

After you remove the failed next hop, the remaining next hops replace it. This prevents impact to any flows that hash to working next hops.

Resilient hashing does not prevent possible impact to existing flows when you add new next hops. Because there are a fixed number of buckets, a new next hop requires reassigning next hops to buckets.

{{< img src = "/images/cumulus-linux/ecmp-reshash-add.png" >}}

As a result, some flows hash to new next hops, which can impact anycast deployments.

Cumulus Linux does *not* enable resilient hashing by default. When you enable resilient hashing, all ECMP groups share 65,536 buckets. An ECMP group is a list of unique next hops that multiple ECMP routes reference.

An ECMP route counts as a single route with multiple next hops:

```
cumulus@switch:~$ ip route show 10.1.1.0/24
10.1.1.0/24  proto zebra  metric 20
  nexthop via 192.168.1.1 dev swp1 weight 1 onlink
  nexthop via 192.168.2.1 dev swp2 weight 1 onlink
```

All ECMP routes must use the same number of buckets (you cannot configure the number of buckets per ECMP route).

{{%notice note%}}
A larger number of ECMP buckets reduces the impact on adding new next hops to an ECMP route. However, the system supports fewer ECMP routes. If you install the maximum number of ECMP routes, new ECMP routes log an error and do not install.

You can configure route and MAC address hardware resources depending on ECMP bucket size changes. See {{%link title="Routing#NVIDIA Spectrum Switches" text="NVIDIA Spectrum routing resources" %}}.
{{%/notice%}}

To enable resilient hashing:

{{< tabs "TabID384 ">}}
{{< tab "NVUE Commands ">}}

There are no NVUE commands for resilient hashing.

{{< /tab >}}
{{< tab "Linux Commands ">}}

1. Edit the `/etc/cumulus/datapath/traffic.conf` file to uncomment and set the `resilient_hash_enable` parameter to `TRUE`.

   You can also set the `resilient_hash_entries_ecmp` parameter to the number of hash buckets to use for all ECMP routes. On Spectrum switches, you can set the number of buckets to 64, 512, 1024, 2048, or 4096. On NVIDIA Spectrum-2 and later, you can set the number of buckets to 64, 128, 256, 512, 1024, 2048, or 4096. The default value is 64.

   ```
   # Enable resilient hashing
   resilient_hash_enable = TRUE

   # Resilient hashing flowset entries per ECMP group
   # 
   # Mellanox Spectrum platforms:
   # Valid values - 64, 512, 1024, 2048, 4096
   #
   # Mellanox Spectrum2/3 platforms
   # Valid values -  64, 128, 256, 512, 1024, 2048, 4096
   #
   # resilient_hash_entries_ecmp = 64
   ```

2. {{<link url="Configuring-switchd#restart-switchd" text="Restart">}} the `switchd` service:
<!-- vale off -->
{{<cl/restart-switchd>}}
<!-- vale on -->
{{< /tab >}}
{{< /tabs >}}

## Adaptive Routing

Adaptive routing is a load balancing mechanism that improves network utilization by selecting routes dynamically based on the immediate network state, such as switch queue length and port utilization.

Cumulus Linux only supports adaptive routing with:
- Switches on Spectrum-2 and later
- {{<link url="RDMA-over-Converged-Ethernet-RoCE" text="RoCE" >}}
- Unicast traffic
- Physical uplink (layer 3) ports; you *cannot* configure adaptive routing on subinterfaces or on ports that are part of a bond
- Interfaces in the default VRF

{{%notice note%}}
Adaptive routing does not make use of resilient hashing.
{{%/notice%}}

Adaptive Routing is in Sticky Free mode, which uses a periodic grades-based egress port selection process.
- The grade on each port, which is a value between 0 and 4, depends on buffer usage and link utilization. A higher grade, such as 4, indicates that the port is more congested or that the port is down. Each packet routes to the less loaded path to best utilize the fabric resources and avoid congestion. The adaptive routing engine always selects the least congested port (with the lowest grade). If there are multiple ports with the same grade, the engine randomly selects between them.
- The change decision for port selection is set to one microsecond; you cannot change it.

{{%notice note%}}
You must configure adaptive routing on *all* ports that are part of the same ECMP route. Make sure the ports are physical uplink ports.
{{%/notice%}}

To enable adaptive routing:

{{< tabs "TabID421 ">}}
{{< tab "NVUE Commands ">}}

For each port on which you want to enable adaptive routing, run the `nv set interface <interface> router adaptive-routing enable on` command.

```
cumulus@switch:~$ nv set interface swp51 router adaptive-routing enable on
cumulus@switch:~$ nv config apply
```

When you run the above command, NVUE:
- Enables the adaptive routing feature.
- Sets adaptive routing on the specified port.
- Sets the link utilization threshold percentage to the default value of 70.

To change the link utilization threshold percentage, run the `nv set interface <interface> router adaptive-routing link-utilization-threshold` command. You can set a value between 1 and 100.

```
cumulus@switch:~$ nv set interface swp51 router adaptive-routing link-utilization-threshold 100
cumulus@switch:~$ nv config apply
```

To disable adaptive routing globally, run the `nv set router adaptive-routing enable off` command. To disable adaptive routing on a port, run the `nv set interface <interface> router adaptive-routing enable off` command.

{{< /tab >}}
{{< tab "Linux Commands ">}}

1. Edit the `/etc/cumulus/switchd.d/adaptive_routing.conf` file:
   - Set the global `adaptive_routing.enable` parameter to `TRUE`.
   - For each port on which you want to enable adaptive routing, set the `interface.<port>.adaptive_routing.enable` parameter to `TRUE`.
   - For each port on which you want to enable adaptive routing, set the `interface.<port>.adaptive_routing.link_util_thresh` parameter to configure the link utilization threshold percentage (optional). You can set a value between 1 and 100. The default value is 70.

```
cumulus@switch:~$ sudo nano /etc/cumulus/switchd.d/adaptive_routing.conf
## Global adaptive-routing enable/disable setting 
adaptive_routing.enable = TRUE

## Supported AR profile modes : STICKY_FREE
#adaptive_routing.profile0.mode = STICKY_FREE

## Maximum value for buffer-congestion threshold is 16777216. Unit is in cells
#adaptive_routing.congestion_threshold.low = 100
#adaptive_routing.congestion_threshold.medium = 1000
#adaptive_routing.congestion_threshold.high = 10000

## Per-port configuration for adaptive-routing
interface.swp51.adaptive_routing.enable = TRUE
interface.swp51.adaptive_routing.link_util_thresh = 70
...
```

The `/etc/cumulus/switchd.d/adaptive_routing.conf` file contains the adaptive routing profile mode (`STICKY_FREE`) and *default* buffer congestion threshold settings; do not change these settings.

2. {{<link url="Configuring-switchd#restart-switchd" text="Restart">}} the `switchd` service:
<!-- vale off -->
{{<cl/restart-switchd>}}
<!-- vale on -->

To disable adaptive routing globally, set the `adaptive_routing.enable` parameter to `FALSE` in the `/etc/cumulus/switchd.d/adaptive_routing.conf` file.

To disable adaptive routing on a port, set the `interface.<port>.adaptive_routing.enable` parameter  to `FALSE` in the `/etc/cumulus/switchd.d/adaptive_routing.conf` file.

{{< /tab >}}
{{< /tabs >}}
<!-- vale off -->
## cl-ecmpcalc
<!-- vale on -->
Run the `cl-ecmpcalc` command to determine a hardware hash result. For example, you can see which path a flow takes through a network. You must provide all fields in the hash, including the ingress interface, layer 3 source IP, layer 3 destination IP, layer 4 source port, and layer 4 destination port.

{{%notice note%}}
`cl-ecmpcalc` only supports input interfaces that convert to a single physical port in the port tab file, such as the physical switch ports (swp). You can not specify virtual interfaces like bridges, bonds, or subinterfaces.
{{%/notice%}}

```
cumulus@switch:~$ sudo cl-ecmpcalc -i swp1 -s 10.0.0.1 -d 10.0.0.1 -p tcp --sport 20000 --dport 80
ecmpcalc: will query hardware
swp3
```

If you omit a field, `cl-ecmpcalc` fails.

```
cumulus@switch:~$ sudo cl-ecmpcalc -i swp1 -s 10.0.0.1 -d 10.0.0.1 -p tcp
ecmpcalc: will query hardware
usage: cl-ecmpcalc [-h] [-v] [-p PROTOCOL] [-s SRC] [--sport SPORT] [-d DST]
                   [--dport DPORT] [--vid VID] [-i IN_INTERFACE]
                   [--sportid SPORTID] [--smodid SMODID] [-o OUT_INTERFACE]
                   [--dportid DPORTID] [--dmodid DMODID] [--hardware]
                   [--nohardware] [-hs HASHSEED]
                   [-hf HASHFIELDS [HASHFIELDS ...]]
                   [--hashfunction {crc16-ccitt,crc16-bisync}] [-e EGRESS]
                   [-c MCOUNT]
cl-ecmpcalc: error: --sport and --dport required for TCP and UDP frames
```

When there are multiple routes in the routing table, Cumulus Linux assigns each route to an ECMP *bucket*. When the ECMP hash executes, the result of the hash determines which bucket to use.

In the following example, four next hops exist. Three different flows hash to different hash buckets. Each next hop goes to a unique hash bucket.

{{< img src = "/images/cumulus-linux/ecmp-hash-bucket.png" >}}

The addition of a next hop creates a new hash bucket. The assignment of next hops to hash buckets, as well as the hash result, sometimes changes with the addition of next hops.

{{< img src = "/images/cumulus-linux/ecmp-hash-bucket-added.png" >}}

With the addition of a new next hop, there is a new hash bucket. As a result, the hash and hash bucket assignment changes, so the existing flows go to different next hops.

When you remove a next hop, the remaining hash bucket assignments can change, which can also change the next hop selected for an existing flow.

{{< img src = "/images/cumulus-linux/ecmp-hash-failure.png" >}}

{{< img src = "/images/cumulus-linux/ecmp-hash-post-failure.png" >}}

A next hop fails, which removes the next hop and hash bucket. It is possible that Cumulus Linux reassigns the remaining next hops.

In most cases, modifying hash buckets has no impact on traffic flows as the switch forwards traffic to a single end host. In deployments where multiple end hosts use the same IP address (anycast), you must use *resilient hashing*.

## Considerations

When the router adds or removes ECMP paths, or when the next hop IP address, interface, or tunnel changes, the next hop information for an IPv6 prefix can change. [FRR](## "FRRouting") deletes the existing route to that prefix from the kernel, then adds a new route with all the relevant new information. In certain situations, Cumulus Linux does not maintain resilient hashing for IPv6 flows.

To work around this issue, you can enable IPv6 route replacement.

{{%notice info%}}
For certain configurations, IPv6 route replacement can lead to incorrect forwarding decisions and lost traffic. For example, it is possible for a destination to have next hops with a gateway value with the outbound interface or just the outbound interface itself, without a gateway address. If both types of next hops for the same destination exist, route replacement does not operate correctly; Cumulus Linux adds an additional route entry and next hop but does not delete the previous route entry and next hop.
{{%/notice%}}

To enable the IPv6 route replacement option:

1. In the `/etc/frr/daemons` file, add the configuration option `--v6-rr-semantics` to the zebra daemon definition. For example:

    ```
    cumulus@switch:~$ sudo nano /etc/frr/daemons
    ...
    vtysh_enable=yes
    zebra_options=" -M cumulus_mlag -M snmp -A 127.0.0.1 --v6-rr-semantics -s 90000000"
    bgpd_options=" -M snmp  -A 127.0.0.1"
    ospfd_options=" -M snmp -A 127.0.0.1"
    ...
    ```
<!-- vale off -->
2. {{<cl/restart-frr>}}
<!-- vale on -->
To verify that IPv6 route replacement, run the `systemctl status frr` command:

```
cumulus@switch:~$ systemctl status frr

    ● frr.service - FRRouting
      Loaded: loaded (/lib/systemd/system/frr.service; enabled; vendor preset: enabled)
      Active: active (running) since Mon 2020-02-03 20:02:33 UTC; 3min 8s ago
        Docs: https://frrouting.readthedocs.io/en/latest/setup.html
      Process: 4675 ExecStart=/usr/lib/frr/frrinit.sh start (code=exited, status=0/SUCCESS)
      Memory: 14.4M
      CGroup: /system.slice/frr.service
            ├─4685 /usr/lib/frr/watchfrr -d zebra bgpd staticd
            ├─4701 /usr/lib/frr/zebra -d -M snmp -A 127.0.0.1 --v6-rr-semantics -s 90000000
            ├─4705 /usr/lib/frr/bgpd -d -M snmp -A 127.0.0.1
            └─4711 /usr/lib/frr/staticd -d -A 127.0.0.1
```
