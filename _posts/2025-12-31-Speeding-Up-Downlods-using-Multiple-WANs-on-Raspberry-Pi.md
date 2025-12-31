---
title: 'Speeding Up Downloads using Multiple WANs on Raspberry Pi'
date: 2025-12-31
permalink: /posts/2025/12/blog-download-speedup/
tags:
  - Infrastructure
---

# Background
We may often find ourselves in situations needing to download large files. However, our download link may not always have enough bandwidth, or we could be rate-limited. Wireless@SGx, for example, is widely used but has a limit of 5Mbps (although practically it could reach 30Mbps).

# Idea
Large downloads are often the result of having to download multiple files in aggregate. For example, a model checkpoint download often contains multiple `.safetensor` files. If we have multiple authenticated WANs on a router, we should be able to load balance the individual downloads among these WANs. This should allow us to utilize the total bandwidth of all the WAN links, give or take.

# Requirements
In this example, I am utilizing two WANs links. One link will use the onboard Wifi chipset. Another link will use USB tethering from a phone. Modify accordingly if you increase number of WANs / use Ethernet etc.
1. Raspberry Pi ( I use RPi 5 )
2. [OpenWRT Factory Image for Raspberry Pi](https://openwrt.org/toh/raspberry_pi_foundation/raspberry_pi#installation)
3. SD Card
4. Developer Laptop
5. Ethernet Cable
6. Mobile Phone for USB Tethering

# Setup
1. Follow the [instructions](https://openwrt.org/toh/raspberry_pi_foundation/raspberry_pi#how_to_flash_openwrt_to_an_sd_card) to flash an SD card with the OpenWRT Factory Image.
2. Use Ethernet Cable to connect your Laptop to the Raspberry Pi. 
3. Access Router Configuration Page at `192.168.1.1` through a browser.
4. Access the wireless page and authenticate to a Wifi hotspot. Your OpenWRT router should now have access to Internet.
5. Access Router over `ssh root@192.168.1.1`
6. `opkg update && opkg install kmod-usb-net-rndis kmod-usb-net-cdc-ether usbutils `
7. Connect phone and activate USB tethering. On the Admin page, add a new DHCP client with the usb0 interface. Assign it to LAN firewall zone.
![Create USB Interface OpenWRT](https://cnboonhan.github.io/files/blog/blog-download-speedup/create-usb-interface-openwrt.png)
![Firewall Zone for USB Interface](https://cnboonhan.github.io/files/blog/blog-download-speedup/usb-interface-firewall-zone-openwrt.png)

8. ---
title: 'Speeding Up Downloads using Multiple WANs on Raspberry Pi'
date: 2025-12-31
permalink: /posts/2025/12/download-speedup/
tags:
  - Infrastructure
---

We may often find ourselves in situations needing to download large files. However, our download link may not always have enough bandwidth, or we could be rate-limited. Wireless@SGx, for example, is widely used but has a limit of 5Mbps (although practically it could reach 30Mbps). How could we improve this?

# Idea
Large downloads are often the result of having to download multiple files in aggregate. For example, a model checkpoint download often contains multiple `.safetensor` files. If we have multiple authenticated WANs on a router, we should be able to load balance the individual downloads among these WANs.

# Requirements
In this example, I am utilizing two WANs links. One link will use the onboard Wifi chipset. Another link will use USB tethering from a phone. Modify accordingly if you increase number of WANs / use Ethernet etc.
1. Raspberry Pi ( I use RPi 5 )
2. [OpenWRT Factory Image for Raspberry Pi](https://openwrt.org/toh/raspberry_pi_foundation/raspberry_pi#installation)
3. SD Card
4. Developer Laptop
5. Ethernet Cable

# Setup
1. Follow the [instructions](https://openwrt.org/toh/raspberry_pi_foundation/raspberry_pi#how_to_flash_openwrt_to_an_sd_card) to flash an SD card with the OpenWRT Factory Image.
2. Use Ethernet Cable to connect your Laptop to the Raspberry Pi.
3. Access Router Configuration Page at `192.168.1.1` through a browser.
4. Access the wireless page and authenticate to a Wifi hotspot. Your OpenWRT router should now have access to Internet.
5. Access Router over `ssh root@192.168.1.1`
6. `opkg update && opkg install kmod-usb-net-rndis kmod-usb-net-cdc-ether usbutils luci-app-mwan3`
7. On the Admin page, add a new DHCP client with the `usb0` interface. Assign it to `wwan` firewall zone. Ensure that a gateway metric is set for the `usb0` and `wwan` interfaces. Set slightly different values for each (`usb0` to `20`, `wwan` to `10`, for example.)
![Create USB Interface OpenWRT](https://cnboonhan.github.io/files/blog/blog-download-speedup/create-usb-interface-openwrt.png)
![Firewall Zone for USB Interface](https://cnboonhan.github.io/files/blog/blog-download-speedup/usb-interface-firewall-zone-openwrt.png)
![alt text](https://cnboonhan.github.io/files/blog/blog-download-speedup/set-gateway-metric-openwrt.png)
8. Configure the MultiWAN Manager under `cgi-bin/luci/admin/network/mwan3` with the following:
![alt text](https://cnboonhan.github.io/files/blog/blog-download-speedup/interfaces-openwrt.png)
![alt text](https://cnboonhan.github.io/files/blog/blog-download-speedup/multiwan-globals-openwrt.png)
![alt text](https://cnboonhan.github.io/files/blog/blog-download-speedup/multiwan-interfaces-openwrt.png)
![alt text](https://cnboonhan.github.io/files/blog/blog-download-speedup/multiwan-members-openwrt.png)
![alt text](https://cnboonhan.github.io/files/blog/blog-download-speedup/multiwan-policies-openwrt.png)
![alt text](https://cnboonhan.github.io/files/blog/blog-download-speedup/multiwan-rules-openwrt.png)
9. Run on SSH terminal: `mwan3 status`. Verify that both WANs are online, and the load balancing ratio is 50/50.
![alt text](https://cnboonhan.github.io/files/blog/blog-download-speedup/multiwan-status-openwrt.png)
10. Run on SSH terminal multiple times: `wget -qO- https://ipecho.net/plain ; echo`
![alt text](https://cnboonhan.github.io/files/blog/blog-download-speedup/load-balancing-result-openwrt.png)
