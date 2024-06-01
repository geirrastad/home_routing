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
- 2 x USB Ethernet interfaces (USB-A 3.0)
- OpenWRT firmware for Raspberry PI 4 (5)
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

## Raspberry Pi - Initial steps

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


