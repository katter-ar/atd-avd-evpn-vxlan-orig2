# EVPN VXLAN Labs

The goal of these labs are to show how AVD eases Day 2 and beyond network operations. 

## Lab 1 - Build Host Connectivity

This lab will show you how to build the host connectivity in both data centers. Once the connectivity is established, the remaining labs will build the VLANs and VRFs that the hosts will be associated to.  During this lab, you will modify the `dc1_fabric_ports.yml` and `dc2_fabric_ports.yml` vars files to add the new host connectivity into the fabric.  

Modify the ***fabric_ports*** vars files to add the host connectivity in.

For dc1, you will modify: `dc1_fabric_ports.yml`

```yaml
 PP-SERVER-112:
   mode: access
   vlans: "112"
 PP-SERVER-134:
   mode: access
   vlans: "134"

servers:
 HOSTA:
   adapters:
     - endpoint_ports: [Eth1, Eth2]
       switch_ports: [Ethernet4, Ethernet4]
       switches: [s1-leaf1, s1-leaf2]
       profile: PP-SERVER-112
       port_channel:
         mode: active
 HOSTB:
   adapters:
     - endpoint_ports: [Eth1, Eth2]
       switch_ports: [Ethernet4, Ethernet4]
       switches: [s1-leaf3, s1-leaf4]
       profile: PP-SERVER-134
       port_channel:
         mode: active

```

For dc2, you will modify: `dc2_fabric_ports.yml`

```yaml
 PP-SERVER-112:
   mode: access
   vlans: "112"
 PP-SERVER-134:
   mode: access
   vlans: "134"

servers:
 HOSTA:
   adapters:
     - endpoint_ports: [Eth1, Eth2]
       switch_ports: [Ethernet4, Ethernet4]
       switches: [s2-leaf1, s2-leaf2]
       profile: PP-SERVER-112
       port_channel:
         mode: active
 HOSTB:
   adapters:
     - endpoint_ports: [Eth1, Eth2]
       switch_ports: [Ethernet4, Ethernet4]
       switches: [s2-leaf3, s2-leaf4]
       profile: PP-SERVER-134
       port_channel:
         mode: active  

```

## Lab 2 - Add VLANs to EVPN VXLAN Topology

This lab will show you how simple it is to automate the configuration changes associated with adding VLANs to your EVPN VXLAN topology.  In a non-automated EVPN VXLAN topology, anytime a new VLAN needs to be added and extended, it has to be created on every switch, added to VXLAN, and added to the BGP configuration on the relevant devices.  During this lab, you will modify the `dc1_fabric_services.yml` and `dc2_fabric_services.yml` vars files to add the new VLANs listed below.  

Modify the ***fabric_services*** vars files to add the VLAN 112 for host1 connectivity.

For both DCs, you will modify: `dc1_fabric_services.yml` and  `dc1=2_fabric_services.yml`

```yaml
          112:
            name: onetwelve
            description: onetwelve
            tags: ['DC']
            enabled: true
            mtu: 9014
            ip_address_virtual: 10.111.112.1/24
            evpn_l2_multi_domain: true

```

After modifying and saving the vars files, complete the following steps:

1) Issue the `make build_dc1` and `make build_dc2` to generate the new structured and device configurations.

2) Review the configurations in their respective directories and verify the changes are correct.

3) Review the changes to the documentation that is auto-created.

4) Issue the `make deploy_dc1_eapi` and `make deploy_dc2_eapi`

5) Login to leaf switches 1 and 2 and verify the new configurations are present.

6) Validate that s1-host1 and s2-host1 can ping 10.111.112.1 (s1-host2 and s2-host2 will be validated once their leaf pairs are created)

7) Validate that s1-host1 can ping s2-host1

## Lab 3 - Add Leaf Pairs 3 and 4

This lab will show you the simplicity in adding additional leaf pairs to your existing Layer 3 leaf/spine topology.  In a non-automated topology, when deploying new leafs, you would need to duplicate or create the base configs for the new leafs, and then modify the configurations of the existing devices to add them into the EVPN VXLAN overlay.  For this lab, you will be adding leafs 3 and 4 in each datacenter into the network topology by following the below steps.

1) The first step is to add the new leafs into their respective inventory files:

For dc1, you will modify:  `sites/dc1/inventory.yml`

Add the following switches into the correct location in the file:

```yaml
            s1-leaf3:
            s1-leaf4:
```

For dc2, you will modify:  `sites/dc2/inventory.yml`

Add the following switches into the correct location in the file:

```yaml
            s2-leaf3:
            s2-leaf4:
```

2) To enable AVD to generate all the required configuration changes, you will only need to modify the `*_fabric.yml` files for each datacenter.  Follow the  YAML file structure for the existing leaf pairs, 1 and 2, and enter the required changes using the parameters below:

For dc1, you will modify:  `sites/dc1/group_vars/dc1_fabric.yml`

Use the following parameters for Leafs 3 and 4, which are leaf pair 2.

```yaml
    - group: LeafPair2
      filter:
        tenants: [ATD_DC]
        tags: ['DC']
      bgp_as: 65002
      nodes:
        - name: s1-leaf3
          id: 18
          mgmt_ip: 192.168.0.14/24
          uplink_switch_interfaces: [Ethernet4, Ethernet4]
        - name: s1-leaf4
          id: 20
          mgmt_ip: 192.168.0.15/24
          uplink_switch_interfaces: [Ethernet5, Ethernet5]

```

For dc2, you will modify:  `sites/dc2/group_vars/dc2_fabric.yml`

Use the following parameters for Leafs 3 and 4, which are leaf pair 2.

```yaml
    - group: LeafPair2
      filter:
        tenants: [ATD_DC]
        tags: ['DC']
      bgp_as: 65102
      nodes:
        - name: s2-leaf3
          id: 118
          mgmt_ip: 192.168.0.24/24
          uplink_switch_interfaces: [Ethernet4, Ethernet4]
        - name: s2-leaf4
          id: 120
          mgmt_ip: 192.168.0.25/24
          uplink_switch_interfaces: [Ethernet5, Ethernet5]

```
## Lab 4 - Add VRFs and VLANs to EVPN VXLAN Topology

This lab will show you how simple it is to automate the configuration changes associated with adding additional VRFs to your EVPN VXLAN topology.  In a non-automated EVPN VXLAN topology, anytime a new VRF needs to be added and extended, it has to be created on every switch, added to VXLAN, and added to the BGP configuration on the relevant devices.  During this lab, you will modify the `dc1_fabric_services.yml` and `dc2_fabric_services.yml` vars files to add the new VRFs and associated VLANs listed below.  

Modify the ***fabric_services*** vars files to add the VRF B and VLAN 134 for host2 connectivity:

For both DCs, you will modify: `dc1_fabric_services.yml` and  `dc1=2_fabric_services.yml`

```yaml
      B:
        vrf_vni: 50002
        mlag_ibgp_peering_vlan: 4002
        mlag_ibgp_peering_ipv4_pool: 192.3.2.0/23
        svis:
          134:
            name: onethirtyfour
            description: onethirtyfour
            tags: ['DC']
            enabled: true
            mtu: 9014
            ip_address_virtual: 10.111.134.1/24
            evpn_l2_multi_domain: true

```

After modifying and saving the vars files, complete the following steps:

1) Issue the `make build-dc1` and `make build-dc2` to generate the new structured and device configurations.
2) Review the configurations in their respective directories and verify the changes are correct.  
3) Review the changes to the documentation that is auto-created.
4) Issue the `make deploy_dc1_eapi` and `make deploy_dc2_eapi`
5) Issue the `make bounce` to reset err-disabled interfaces ethernet 2 & 3 for s1-leaf3&4 and s2-leaf3&4.
6) Validate that s1-host2 and s2-host2 can ping 10.111.134.1
7) Validate that s1-host2 can ping s2-host2
8) Validate that s1-host1 CANNOT ping s1-host2
9) Validate that s2-host1 CANNOT ping s2-host2

