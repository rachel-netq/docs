---
title: NetQ Release Versioning and Support Policy
author: Cumulus Networks
weight: 703
toc: 4
---

This article outlines the release version numbering structure and support policies for NetQ.
## Version Definitions

NetQ software installation file names include a version number, in the form of x.y.z-OS\~TAG\_CPU.

- **x** represents the major release version number. An increased major release version means that the release might include:
    - New functionality within the existing market.
    - New market entries.
    - Major positioning changes.
    - A significant engineering rebase.
- **y** represents the minor release version number. An increased minor release version means that the release might include:
    - New hardware platforms.
    - New features within the existing market.
    - Bug fixes.
    - Security updates.
- **z** represents the maintenance release version number. An increased maintenance release version might include:
    - Feature improvements.
    - Bug fixes and updates.
    - Security updates.
- **OS** indicates the Network Operating System information (cl for Cumulus Linux, deb10 for SONiC)
- **TAG** represents a timestamp for the release of the version.
- **CPU** architecture represents architecture.

For example:

- netq-agent\_4.1.0-cl4u36~1639165028.2c18daee\_amd64.deb
- netq-apps\_4.1.0-cl4u36~1639165028.2c18daee\_amd64.deb

## Release, Support Lifecycle and Support Policy

NetQ is offered with a per switch subscription that includes support for 1, 3, and 5 years options. The subscription model allows customers to upgrade the software as updates and new versions become available, for the period of the subscription.

- Review the NetQ [user guide]({{<ref "/cumulus-netq-41/" >}}) for the supported Network Operating System (NOS) versions.
- Use matching versions of NetQ Server and both NetQ Agent and NetQ Apps packages on switches (e.g., NetQ 4.1.0 Server with NetQ 4.1.0 Agents and Apps on the switches).
- The product is supported for the period of the subscription and bug fixes are received by upgrading to new versions of software.
- A NetQ version is supported for two years from its release date. After that date, it is necessary to upgrade to a later release to continue receiving support for the period of the subscription.

## NetQ Support Matrix

The following table depicts the NetQ release support matrix:

| NetQ Release | Release Date | End of Support |
| :--------: | --------- | --------- |
| 4.1.z | 13-Jan-2022 | 13-Jan-2024 |
| 4.0.z| 15-Sep-2021 | 15-Sep-2023 |
| 3.y | 21-Mar-2021 | 21-Mar-2023 |

**NetQ 1.y and 2.y releases are End of Support.**

## Upgrade Process

For information regarding upgrading from previous NetQ releases, refer to the [NetQ Deployment Guide]({{<ref "/cumulus-netq-41/Manage-Deployment" >}}).
