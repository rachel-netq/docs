---
title: Uninstall the ONYX Image
author: Nvidia
weight: 252
toc: 4
---

If you want to migrate from {{<exlink url="https://www.nvidia.com/en-us/networking/ethernet-switching/onyx/" text="ONYX">}} to Cumulus Linux, you must uninstall the ONYX image from the switch before you install Cumulus Linux.

Follow this procedure:

1. Log in to the ONYX switch, then run the `reload` command to reboot the switch.

   {{< img src = "/images/knowledge-base/uninstall-onyx/onyx-reload.png" >}}

2. When the switch boots, select `ONIE: Uninstall OS` from the ONIE GRUB menu and press `Enter`.

   {{< img src = "/images/knowledge-base/uninstall-onyx/onie-uninstall.png" >}}
   
   {{%notice note%}}
On older ONYX releases, called *MLNX OS*, you might see a different GRUB menu. In this case, select `ONIE` and press `Enter`.

{{< img src = "/images/knowledge-base/uninstall-onyx/old-onyx.png" >}}

To confirm that you want to uninstall the MLNX network operating system, type `YES` at the prompt, then press `Enter`. This process can take up to 90 minutes.

{{< img src = "/images/knowledge-base/uninstall-onyx/old-onyx2.png" >}}
{{%/notice%}}

3. To confirm that you want to uninstall the ONYX network operating system, type `YES` at the prompt, then press `Enter`.

   {{< img src = "/images/knowledge-base/uninstall-onyx/onie-yes.png" >}}

4. The uninstallation process starts, which usually takes several seconds but can take a few minutes. At the end of the process, the switch reboots automatically, then boots into ONIE again.

   {{< img src = "/images/knowledge-base/uninstall-onyx/onie-uninstall-start.png" >}}

5. Select `ONIE: Install OS` and press `Enter`.

   {{< img src = "/images/knowledge-base/uninstall-onyx/onie-install-os.png" >}}

6. After the switch enters ONIE: OS Install Mode, the ONIE Service Discovery process starts.

   {{< img src = "/images/knowledge-base/uninstall-onyx/install mode.png" >}}

   If you need to stop the process, press `Enter` and run the `onie-discovery-stop` command.

   {{< img src = "/images/knowledge-base/uninstall-onyx/discovery stop.png" >}}

7. Install the Cumulus Linux image. Refer to [Installing a New Cumulus Linux Image]({{<ref "/cumulus-linux-50/Installation-Management/Installing-a-New-Cumulus-Linux-Image.md" >}}).

{{%notice info%}}
If the switch ships with ONYX from the factory, the platform identifier (PSID) code restricts the switch to ONYX and the switch only supports NVIDIA optics. When migrating from ONYX to Cumulus Linux, you must change the PSID to remove this lock and enable all non-NVIDIA optics.
Future releases plan to change the PSID automatically. For more information, contact NVIDIA support.
{{%/notice%}}

To ease the transition from ONYX to NVUE, you can upload the running configuration file from the ONYX switch with this {{<exlink url="https://air.nvidia.com/migrate/" text="NVUE Migration Tool">}}.
