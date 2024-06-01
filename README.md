# home_routing
Home-route your internet traffic.

## What does that mean?

It means that all your internet bound traffic is routed via your home network instead of directly from your computer or your current network.
If you are abroad, or have a second ISP in your summer house etc, this mechanism will make your traffic originate from your home network.

## Why would I do that ?

1. Security - You might be in a spot where man-in-the-middle attacks can happen. This will encrypt traffic from your computer.
2. National services - Some TV stations or newspapers etc will only serve traffic from your home country
3. Streaming services - Big content provideers (rhymes with Vetmix ...) suddenly requires all nodes to be on the same network. Even if you paid for 4 separate streams, and was told you could stream anytime from anywhere, this is no longer the truth. The devices must be on the same network.

Bullet 3 will not open up for massive subsription sharing, as you would NOT let stranger into your home  network

It is poorly hidden that my main reason to do home routing is because of number 3 :)

## What do I need ?

This version will use ZeroTier, and two rasopberry pies running openwrt.You could do this with other systems, and pure routing / iptables, but OpenWRT made it sooo simple.
I have a setup that differs slightly from this. I use VLANs so I do not need the second ethernet interface on the raspis. I also have a separate VLAN so home routing traffic does not go via my internal 192.x.x.x address and isolates my connected devices at home.

- 2 x Raspberry PI (4 or 5)
- 1 x USB Ethernet interfaces (USB-A 3.0)
- OpenWRT firmware for Raspberry PI 4 (5) (installed on the "client" side)
- ZeroTier account (it is free for up to 25 nodes)

## ZeroTier - Initial step

1. Go over to [ZeroTier](https://www.zerotier.com/) and register for an account
2. Log in with your account and press "Create A Network"
3. Click on the newly created network (It got a random name from ZeroTier but can be changed)

### Basics
  Give it a new name, and optionally add a description. Keep it private.
   
### Advanced

First select a IP Range. In this example I picked the 10.147.17.*. This will give you plenty of address space.
Make sure the "Auto-Assign from Range" is selected
Leave all other options as they are

Write down or keep the "Network ID" (at the top) handy, you will need it later

## Exit node: RaspbianOS / Ubuntu

The exit node (the one located in your home network). Only needs basic OS and the ZeroTier client

1. Flash any OS on your device
2. Connect it to your home LAN
3. Boot it up and upgrade packages
4. Download and install ZeroTier

  curl -s 'https://raw.githubusercontent.com/zerotier/ZeroTierOne/main/doc/contact%40zerotier.com.gpg' | gpg --import && \  
if z=$(curl -s 'https://install.zerotier.com/' | gpg); then echo "$z" | sudo bash; fi


5. Enable zerotier-one service

  sudo service zerotier enable
  sudo service zerotier start

6. Join your private ZT network

  sudo zerotier-cli join <network id from zerotier initial setup>

7. Now log in to your ZeroTier account
8. Click on your network and scroll down to the members section. You should see a red entry
9. Edit that entry: IP Address: 10.147.17.10 and press the +
10. Give it a name "Main home router" and a description
11. Enable it by selecting the "Auth" checkbox
12. Go to your pi and run

  ip addr  (on ubuntu)
  ifconfig (on rasbian)

13. You should now see a zerotier interface with an ip address of 10.147.17.10
14. Now go back to ZeroTier UI and scroll up to the "Advanced" section
15. Add a new route: destination: 0.0.0.0/0 via: 10.147.17.10 then "Submit". This is the route that enables home routing!





## Client Node: OpenWRT - Initial steps

Note: At the time of writing, Raspberry PI 5 support in OpenWRT is not yet released.
- Download OpenWRT [Raspberry PI firmware](https://openwrt.org/toh/raspberry_pi_foundation/raspberry_pi).
- Click on the correct Raspberry Pi version in the "Firmware OpenWRT Install URL" column of the "Installation" table.
- Follow [this guide](https://openwrt.org/toh/raspberry_pi_foundation/raspberry_pi#how_to_flash_openwrt_to_an_sd_card) to flash the firmware.
  NOTE:
    Choose the "Customize installed packages and/or first boot script" and click "Request Build" hwen ready. If you don't do this
    you'll probably have to manually installe those packages to enable luci (admin UI of OpenWRT)
    If you know the chipset of your ethernet USB dongle, you may add that driver in the package list before pressing Request Build.
    Many dongles uses the "kmod-usb-net-rtl8152" driver

### Boot the Pi with openwrt

On first-time boot the OpenWRT uses the address 192.168.1.1. You need to connect the built-in ETH port to your computer. The computer needs a static IP: 192.168.1.2 (or any other address in the 192.168.1.x range. except 192.168.1.1 which is the Pi)

Open a webbrowser on http://192.168.1.1 and you should see the Luci interface. If you don't, try to put the lan cable into the dongle on the Pi. If it still doesn't work, you'll need to attach a monitor and keyboard to your Pi
and run ifconfig to see if the interface is activated.

If you have a network connection, but no luci interface, try to run this command:

  opkg install base-files bcm27xx-gpu-fw bcm27xx-utils brcmfmac-nvram-43455-sdio busybox ca-bundle cypress-firmware-43455-sdio dnsmasq dropbear e2fsprogs firewall4 fstools iwinfo kmod-brcmfmac kmod-fs-vfat kmod-nft-offload kmod-nls-cp437 kmod-nls-iso8859-1 kmod-r8169 kmod-sound-arm-bcm2835 kmod-sound-core kmod-usb-hid kmod-usb-net-lan78xx libc libgcc libustream-mbedtls logd luci mkf2fs mtd netifd nftables odhcp6c odhcpd-ipv6only opkg partx-utils ppp ppp-mod-pppoe procd procd-seccomp procd-ujail uci uclient-fetch urandom-seed wpad-basic-mbedtls kmod-usb-net-rtl8152

then reboot the Pi.

Assuming you got a LUCI interface on http://192.168.1.1 we will continue. otherwise, google, as it could be a multiple of issues

### Setting up OpenWRT

1. You will have to set a new password for root
2. Go to "Network -> Interfaces"
3. Choose "Add new interface"
4. Call it "WAN", protocol: "DHCP Client", Device: "choose your USB device". Click "Create interface"
  NOTE: You may choose "Static address" but then you will have to set IP address, mask, gateway and DNS manually for this interface
5. Open the interface, and go to "Firewall settings". Make sure it is put in the "WAN" zone

Optional:
6. If your main network already is in the 192.168.1.0/24 range, you will have to change IP on the 


  



0


