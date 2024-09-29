Download: 3.50 Mbps
Upload: 20.05 Mbps

Very Slow Download Speed on Ubuntu 20.04 with a wired connection

### Restart NetworkManager after each change to avoid rebooting:

```bash
sudo systemctl restart NetworkManager
```

### Speed test

```bash
sudo apt  install speedtest-cli
speedtest-cli --secure 
```

### Get info on the current Wi-Fi controller

```bash
lspci | grep Wireless
55:00.0 Wireless controller [0d40]: Intel Corporation XMM7360 LTE Advanced Modem (rev 01)
```

Driver info (`driver=iwlwifi driverversion=6.8.0-45-generic`:
```bash
lspci | grep Wireless
  *-network                 
       description: Wireless interface
       product: Wi-Fi 6 AX201
       vendor: Intel Corporation
       physical id: 14.3
       bus info: pci@0000:00:14.3
       logical name: wlp0s20f3
       version: 20
       serial: 7c:21:4a:aa:ba:68
       width: 64 bits
       clock: 33MHz
       capabilities: pm msi pciexpress msix bus_master cap_list ethernet physical wireless
       configuration: broadcast=yes driver=iwlwifi driverversion=6.8.0-45-generic firmware=77.ad46c98b.0 QuZ-a0-hr-b0-77.u ip=192.168.2.110 latency=0 link=yes multicast=yes wireless=IEEE 802.11
       resources: iomemory:600-5ff irq:16 memory:603f29c000-603f29ffff
```

More detailed driver information:
```bash
lspci -nnv -s 55:00.0
55:00.0 Wireless controller [0d40]: Intel Corporation XMM7360 LTE Advanced Modem [8086:7360] (rev 01)
	Subsystem: Hewlett-Packard Company XMM7360 LTE Advanced Modem [103c:8337]
	Flags: bus master, fast devsel, latency 0, IRQ 190, IOMMU group 16
	Memory at 6e300000 (64-bit, non-prefetchable) [size=4K]
	Memory at 6e301000 (64-bit, non-prefetchable) [size=1K]
	Capabilities: <access denied>
	Kernel driver in use: iosm
	Kernel modules: iosm
modinfo iosm
filename:       /lib/modules/6.8.0-45-generic/kernel/drivers/net/wwan/iosm/iosm.ko.zst
name:           iosm
vermagic:       6.8.0-45-generic SMP preempt mod_unload modversions
```

TODO drivers vs firmware:

```bash
sudo mkdir -p /lib/firmware/ath10k/QCA6174/hw3.0/
sudo wget -O /lib/firmware/ath10k/QCA6174/hw3.0/board.bin https://github.com/FireWalkerX/ath10k-firmware/blob/7e56cbb94182a2fdab110cf5bfeded8fd1d44d30/QCA6174/hw3.0/board-2.bin?raw=true
sudo wget -O /lib/firmware/ath10k/QCA6174/hw3.0/firmware-4.bin https://github.com/FireWalkerX/ath10k-firmware/blob/7e56cbb94182a2fdab110cf5bfeded8fd1d44d30/QCA6174/hw3.0/firmware-4.bin_WLAN.RM.2.0-00180-QCARMSWPZ-1?raw=true
sudo chmod +x /lib/firmware/ath10k/QCA6174/hw3.0/*
```

### Current configuration of Wi-Fi module

```bash
iwconfig
```
Result:
```text
wlp0s20f3  IEEE 802.11  ESSID:"EasyBox-033576"  
          Mode:Managed  Frequency:2.412 GHz  Access Point: 78:94:B4:38:76:D8   
          Bit Rate=54 Mb/s   Tx-Power=22 dBm   
          Retry short limit:7   RTS thr:off   Fragment thr:off
          Power Management:off
          Link Quality=41/70  Signal level=-69 dBm  
          Rx invalid nwid:0  Rx invalid crypt:0  Rx invalid frag:0
          Tx excessive retries:0  Invalid misc:10   Missed beacon:0
```

Get detailed wireless information from a wireless interface
```bash
iwlist wlp0s20f3 scan
```

### `iwlwifi` modules info, including list of possible parameters:

```bash
modinfo iwlwifi
...
filename:       /lib/modules/6.8.0-45-generic/kernel/drivers/net/wireless/intel/iwlwifi/iwlwifi.ko.zst
license:        GPL
description:    Intel(R) Wireless WiFi driver for Linux
name:           iwlwifi
vermagic:       6.8.0-45-generic SMP preempt mod_unload modversions
parm:           swcrypto:using crypto in software (default 0 [hardware]) (int)
parm:           11n_disable:disable 11n functionality, bitmap: 1: full, 2: disable agg TX, 4: disable agg RX, 8 enable agg TX (uint)
parm:           amsdu_size:amsdu size 0: 12K for multi Rx queue devices, 2K for AX210 devices, 4K for other devices 1:4K 2:8K 3:12K (16K buffers) 4: 2K (default 0) (int)
parm:           fw_restart:restart firmware in case of error (default true) (bool)
parm:           nvm_file:NVM file name (charp)
parm:           uapsd_disable:disable U-APSD functionality bitmap 1: BSS 2: P2P Client (default: 3) (uint)
parm:           enable_ini:0:disable, 1-15:FW_DBG_PRESET Values, 16:enabled without preset value defined,Debug INI TLV FW debug infrastructure (default: 16) (uint)
parm:           bt_coex_active:enable wifi/bt co-exist (default: enable) (bool)
parm:           led_mode:0=system default, 1=On(RF On)/Off(RF Off), 2=blinking, 3=Off (default: 0) (int)
parm:           power_save:enable WiFi power management (default: disable) (bool)
parm:           power_level:default power save level (range from 1 - 5, default: 1) (int)
parm:           disable_11ac:Disable VHT capabilities (default: false) (bool)
parm:           remove_when_gone:Remove dev from PCIe bus if it is deemed inaccessible (default: false) (bool)
parm:           disable_11ax:Disable HE capabilities (default: false) (bool)
parm:           disable_11be:Disable EHT capabilities (default: false) (bool)
```

### List of loaded/current parameters:

```bash
sudo apt install sysfsutils
systool -vm iwlwifi
```
Output:
```text
Module = "iwlwifi"

  Attributes:
    coresize            = "598016"
    initsize            = "0"
    initstate           = "live"
    refcnt              = "1"
    srcversion          = "2553DA4DACB6342A2AD9180"
    taint               = ""
    uevent              = <store method only>

  Parameters:
    11n_disable         = "1"
    amsdu_size          = "0"
    bt_coex_active      = "Y"
    disable_11ac        = "N"
    disable_11ax        = "N"
    disable_11be        = "N"
    enable_ini          = "16"
    fw_restart          = "Y"
    led_mode            = "0"
    nvm_file            = "(null)"
    power_level         = "0"
    power_save          = "N"
    remove_when_gone    = "N"
    swcrypto            = "0"
    uapsd_disable       = "3"
```

### Set parameters

For instance, to set `11n_disable` and `power_save` parameters:
```bash
sudo modprobe iwlwifi 11n_disable=1
sudo modprobe iwlwifi power_save=disable
```

Permanently in `/etc/modprobe.d/iwlwifi.conf` file as:
```bash
echo "options iwlwifi swcrypto=0 11n_disable=1 power_save=disable" >> /etc/modprobe.d/iwlwifi.conf
```


### System logs from network devices

```bash
sudo dmesg -T | grep "wlp0s20f3\|iwlwifi"
```
Note: `wlp0s20f3` - name from `iwconfig`

### Change to preferring IPv4 over IPv6.

System preferring IPv6 over IPv4. To change to preferring IPv4 over IPv6:

```bash
sudo subl /etc/gai.conf
```
Uncomment the last line to:
```text
#
#    For sites which prefer IPv4 connections change the last line to
#
precedence ::ffff:0:0/96  100
```

[Change to preferring IPv4 over IPv6](https://askubuntu.com/a/1232022/458132)

Additionally, you could disable IPv6 Support entirely:
```bash
sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1
sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1
sudo sysctl -w net.ipv6.conf.lo.disable_ipv6=1
```
If the temporary disablement works, make the changes permanent by running the following commands:
```bash
sudo su
echo "#disable ipv6" >> /etc/sysctl.conf
echo "net.ipv6.conf.all.disable_ipv6 = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.default.disable_ipv6 = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.lo.disable_ipv6 = 1" >> /etc/sysctl.conf
```

### Disable Power Management:

```bash
iwconfig
```
Result:
```text
wlp0s20f3  IEEE 802.11  ESSID:"EasyBox-033576"  
          ...
          Power Management:on
```

Disable:
```bash
sudo iwconfig wlp0s20f3 power off
```
or:
```bash
sudo modprobe iwlwifi power_save=disable
```

Restarting the Network Manager Service:
```bash
sudo systemctl restart NetworkManager.service
```

Check that the changes are applied:
```bash
iwconfig
```
Result:
```text
wlp0s20f3  IEEE 802.11  ESSID:"EasyBox-033576"  
          ...
          Power Management:off
```

Check if this setting helped.

### Disable Wi-Fi Power Saving 

Note: It is not the same as `Power Management`!

Disable Wi-Fi Power Saving  via file configuration:
```bash
sudo subl /etc/NetworkManager/conf.d/default-wifi-powersave-on.conf
```
Change:
```text
[connection]
wifi.powersave = 3
```
to:
```text
[connection]
wifi.powersave = 2
```
Restarting the Network Manager Service:
```bash
sudo systemctl restart NetworkManager.service
```

[Extremely slow wifi between Ubuntu 24.04 LTS laptop and Deco M5 mesh access points](https://superuser.com/questions/1843548/extremely-slow-wifi-between-ubuntu-24-04-lts-laptop-and-deco-m5-mesh-access-poin)

### Disable Active-State Power Management (ASPM)

Note: probably is not actual anymore. Use it as the last option, it affects all hardware.

```bash
sudo subl /etc/default/grub
```
add `pcie_aspm=off` in:
```text
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash pcie_aspm=off"
```
and run:
```bash
sudo update-grub
```

`pcie_aspm` is some sort of power management thing which probably puts the network controller to sleep or something.

[Ubuntu 20.04 slow download speed (wired network)](https://askubuntu.com/a/1347459/458132)

### Change `11n_disable` parameter

Different sources recommend different values as a solution. Possible values for changes:

```bash
modinfo iwlwifi
...
name:           iwlwifi
parm:           11n_disable:disable 11n functionality, bitmap: 1: full, 2: disable agg TX, 4: disable agg RX, 8 enable agg TX (uint)
```
Possible values for changes:
```bash
sudo modprobe iwlwifi 11n_disable=8
sudo modprobe iwlwifi 11n_disable=4
sudo modprobe iwlwifi 11n_disable=2
sudo modprobe iwlwifi 11n_disable=1
sudo modprobe iwlwifi 11n_disable=disable
```
In my case `11n_disable=disable` showed the best result


### Remove `backport-iwlwifi-dkms`

It was a reason in certain cases. 
```bash
sudo apt remove backport-iwlwifi-dkms
```
I don't have this package installed

[Remove `backport-iwlwifi-dkms` for performance](https://askubuntu.com/a/1231662/458132)
