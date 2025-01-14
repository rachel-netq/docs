---
title: What's New
author: NVIDIA
weight: 5
toc: 2
---
This document supports the Cumulus Linux 5.1 release, and lists new platforms, features, and enhancements.

- For a list of all the platforms supported in Cumulus Linux 5.1, see the {{<exlink url="www.nvidia.com/en-us/networking/ethernet-switching/hardware-compatibility-list/" text="Hardware Compatibility List (HCL)">}}.
- For a list of open and fixed issues in Cumulus Linux 5.1, see the {{<link title="Cumulus Linux 5.1 Release Notes" text="Cumulus Linux 5.1 Release Notes">}}.
- To upgrade to Cumulus Linux 5.1, follow the steps in {{<link url="Upgrading-Cumulus-Linux">}}.
<!-- vale off -->
## What's New in Cumulus Linux 5.1.0
<!-- vale on -->
Cumulus Linux 5.1.0 supports new platforms, provides bug fixes, and contains several new features and improvements.

### Platforms

- NVIDIA SN4800 (100G Spectrum-3) available for early access
<!--
- NVIDIA SN2201 (100G Spectrum-1)
-->
### New Features and Enhancements

- {{<link url="GRE-Tunneling" text="GRE tunneling">}}
- {{<link url="Equal-Cost-Multipath-Load-Sharing-Hardware-ECMP/#adaptive-routing" text="Adaptive routing with RoCE">}}
- {{<link url="Link-Layer-Discovery-Protocol/#lldp-dcbx-tlvs" text="LLDP DCBX TLVs">}}
- {{<link url="Equal-Cost-Multipath-Load-Sharing-Hardware-ECMP/#gtp-hashing" text="GTP hashing">}}
- {{<link url="In-Service-System-Upgrade-ISSU" text="Warmboot on bonds">}}
- {{<link url="Multi-Chassis-Link-Aggregation-MLAG/#peer-link-consistency-check" text="MLAG peer link consistency check">}}
- {{<link url="Precision-Time-Protocol-PTP" text="PTP on bonds">}}
- {{<link title="Spanning Tree and Rapid Spanning Tree - STP/#bpdu-guard" text="BPDU guard protodown state and reason">}}
- {{<link url="NVUE-Object-Model" text="NVUE">}} enhancements include:
  - {{<link url="Neighbor-Discovery-ND" text="IPv6 ND configuration options">}}
  - {{<link url="NVUE-Snippets/#flexible-snippets" text="Flexible snippets">}}
  - {{<link url="VXLAN-Devices/#automatic-vlan-to-vni-mapping" text="Automatic VLAN to VNI mapping">}}
  - New NVUE commands

{{%notice note%}}
Cumulus Linux 5.1 includes the NVUE object model. After you upgrade to Cumulus Linux 5.1, running NVUE configuration commands replaces the configuration in the applicable configuration files and removes any configuration you add manually or with automation tools like Ansible, Chef, or Puppet. To keep your configuration, you can do one of the following:

- Update your automation tools to use NVUE.
- {{<link url="NVIDIA-User-Experience-NVUE/#configure-nvue-to-ignore-linux-files" text="Configure NVUE to ignore certain underlying Linux files">}} when applying configuration changes.
- Use Linux and FRR (vtysh) commands instead of NVUE for all switch configuration.

Cumulus Linux 3.7, 4.3, and 4.4 releases continue to support NCLU. For more information, contact your NVIDIA Spectrum platform sales representative.
{{%/notice%}}
