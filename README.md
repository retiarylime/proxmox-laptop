# Install Proxmox On Laptop & Wifi Setup
A compiled guide to install proxmox on a laptop without the need of ethernet cable.

### Prerequisites:
1. Laptop with minimum system requirements to run Proxmox
2. Home wifi
3. Two USB flash drive at least 4GB each
4. [Rufus](https://rufus.ie/en/)
5. [Proxmox iso](https://www.proxmox.com/en/downloads/proxmox-virtual-environment/iso "Proxmox iso")
6. Debian packages - [wpa_supplicant](https://packages.debian.org/search?keywords=wpasupplicant "wpa_supplicant"), [rfkill](https://packages.debian.org/search?suite=default&section=all&arch=any&searchon=names&keywords=rfkill), [dnsmasq](https://packages.debian.org/search?suite=default&section=all&arch=any&searchon=names&keywords=dnsmasq), [dnsutils](https://packages.debian.org/search?suite=default&section=all&arch=any&searchon=names&keywords=dnsutils)
_(*dont forget to download  dependencies for each package)_

## Step-by-step
1. Download the proxmox iso and flash into one USB flash drive using Rufus.
2. Download all the required debian packages together with their dependencies. Save all the files into another USB flash drive.
3. Boot into the USB flash drive containing proxmox iso & proceed with installation. 
4. You will automatically reboot into the proxmox ve. Login with root account.
5. You will have no internet connection at this point. Don't worry cuz we gonna setup the wifi interface manually.
6. Insert the second USB flash drive containing the required debian packages. [Mount the USB flash drive](https://linuxconfig.org/howto-mount-usb-drive-in-linux "Mount the USB flash drive") and install all the packages using `dpkg -i *.deb`.
7. `nano /etc/dnsmasq.conf`, at the bottom line add the following line:
```
domain=pv.local
interface=vmbr1
dhcp-range=10.10.100.10,10.10.100.200,24h
dhcp-option=vmbr1,3,10.10.100.1
server=1.1.1.1
server=8.8.8.8
dhcp-leasefile=/var/lib/misc/dnsmasq.leases
```
8. Enable & run the systemd unit for wpa_supplicant `systemctl enable wpa_supplicant@wlp1s0.service&& systemctl start wpa_supplicant@wlp1s0.service`
_*you can change the network interface name_

9. `    nano /etc/network/interfaces` and edit the file as below:

```
    auto lo
    iface lo inet loopback
    
    # change wlp1s0 to your wifi interface name
    auto wlp1s0
    iface wlp1s0 inet dhcp
		wpa-ssid [your wifi ssid]
		wpa-psk [your wifi password]
    
    #comment all the autogenerated proxmox bridge adapter
    #auto vmbr0
    #iface vmbr0 inet static
    #        address 192.168.0.201/24
    #        gateway 192.168.0.1
    #        bridge-ports enp3s0
    #        bridge-stp off
    #        bridge-fd 0
    
    #add new bridge adapter interface 
    auto vmbr1
    iface vmbr1 inet static
            address 10.10.100.1/24
            bridge-ports none
            bridge-stp off
            bridge-fd 0
            post-up echo 1 > /proc/sys/net/ipv4/ip_forward
            post-up iptables -t nat -A POSTROUTING -s '10.10.100.1/24' -o wlan0 -j MASQUERADE
            post-down iptables -t nat -D POSTROUTING -s '10.10.100.1/24' -o wlan0 -j MASQUERADE


```
10. Disable & reenable the wlp1s0 wifi adpater. Disable the autogenerated vmbr0 adapter & enable the new vmbr1.
```
ifdown wlp1s0 && ifdown vmbr0
ifup wlp1s0 && ifup vmbr1
```
11. Now you should be able to connect to the internet. Test `ping 8.8.8.8`

---
**Resources:**
1. Thanks to Qeba & fff0q for the tutorial https://forum.proxmox.com/threads/how-to-set-up-proxmox-ve-7-on-a-laptop-workstation-with-wifi-wlan.102395/post-542466
