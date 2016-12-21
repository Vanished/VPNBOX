# VPN In a BOX
This project will enable you to configure a Raspberry Pi (or similar) to run OpenVPN on your network. 

This solution will enable you to connect devices such as ATV, PS4, Xbox, Smart TV etc (which do not support VPN) to the RPI just by changing the DNS.

You can run multiple RPIs on your network.

Once configured, all you will need to do is plug the RPI into your home router, and change the DNS on the device that you want to stream from.

## Preparation

You will need :

1) A VanishedVPN subscrption (from www.vanishedvpn.com )
2) A RPI (or similar) flashed with the latest version of Raspian
3) You should have flashed the image to your SD card, and powered up the device
4) Log onto the device via ssh

`ssh root@rasberrypi`

##IP Addressing

My home network is setup as follows:

Internet Router: 192.168.1.1
Subnet Mask: 255.255.255.0
Router gives out DHCP range: 192.168.100-200
If your network range is different, that's fine, use your network range instead of mine.

I'm going to give my Raspberry Pi a static IP address of `192.168.1.2` by configuring `/etc/network/interfaces` like so:

`nano /etc/network/interfaces`

```
auto lo
iface lo inet loopback

auto eth0
allow-hotplug eth0
iface eth0 inet static
    address 192.168.1.2
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 8.8.4.4
```

##NTP

Accurate time is important for the VPN encryption to work. If the VPN client's clock is too far off, the VPN server will reject the client.

You shouldn't have to do anything to set this up, the `ntp` service is installed and enabled by default.

Double-check your Pi is getting the correct time from internet time servers with `ntpq -p`, you should see at least one peer with a `+` or a `*` or an `o`, for example:

```
$ ntpq -p
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
-0.time.xxxx.com 104.21.137.30    2 u   47   64    3  240.416    0.366   0.239
+node01.jp.xxxxx 226.252.532.9    2 u   39   64    7  241.030   -3.071   0.852
*t.time.xxxx.net 104.1.306.769    2 u   38   64    7  127.126   -2.728   0.514
+node02.jp.xxxxx 250.9.592.830    2 u    8   64   17  241.212   -4.784   1.398
```
##Setup VPN client

Install the OpenVPN client:
```
sudo apt-get install openvpn
```
Download and uncompress the PIA OpenVPN profiles:
```
wget https://www.privateinternetaccess.com/openvpn/openvpn.zip
sudo apt-get install unzip
unzip openvpn.zip -d openvpn
```

Copy the PIA OpenVPN certificates and profile to the OpenVPN client:
```
sudo cp openvpn/ca.rsa.2048.crt openvpn/crl.rsa.2048.pem /etc/openvpn/
sudo cp openvpn/Japan.ovpn /etc/openvpn/Japan.conf
```
You can use a diffrent VPN endpoint if you like. Note the extension change from ovpn to conf.

Create `/etc/openvpn/login` containing only your username and password, one per line, for example:

`nano /etc/openvpn/login`

```
user12345678
MyGreatPassword
```
