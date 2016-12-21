# VPN in a BOX
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

`ssh pi@<ip address of your pi>`

default password is `raspberry`

`sudo su -` to get into root

##IP Addressing

My home network is setup as follows:

Internet Router: 192.168.1.1
Subnet Mask: 255.255.255.0
Router gives out DHCP range: 192.168.100-200
If your network range is different, that's fine, use your network range instead of mine.

Find a free IP address (check your router), and assign a static IP to the Pi

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
sudo apt-get install openvpn -y
```
Copy the VanishedVPN OpenVPN config file (USA by default):
```
wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=0B_U-Jx4uRplbUHpQYVZ3WC0yaE0' -O /etc/openvpn/vanished.conf
```
Create `/etc/openvpn/login` containing only your username and password (in the welcome email from VanishedVPN), one per line, for example:

`nano /etc/openvpn/login`

```
my_email_is_my_username
MyVanishedVPNpassword
```

Update the config file to authenticate using your new password file
```
cd /etc/openvpn
```
```
nano vanished.conf
```
Change

``` 
auth-user-pass
```
to
```
auth-user-pass /etc/openvpn/login
```
You will also see the address of the PVN server in this config file. By default, it should be
```
usa.vanishedvpn.com
```
If you want to connect to any other VPN, simply change the address to one of the other vanishedvpn servers

##Test the VPN

At this point you should be able to test the VPN actually works:
```
openvpn --config /etc/openvpn/vanished.conf
```

If all is well, you should see something like this 
```
openvpn --config /etc/openvpn/vanished.conf
Wed Dec 21 02:08:18 2016 OpenVPN 2.2.1 arm-linux-gnueabihf [SSL] [LZO2] [EPOLL] [PKCS11] [eurephia] [MH] [PF_INET6] [IPv6 payload 20110424-2 (2.2RC2)] built on Dec  1 2014
Wed Dec 21 02:08:18 2016 WARNING: file '/etc/openvpn/login' is group or others accessible
Wed Dec 21 02:08:18 2016 WARNING: No server certificate verification method has been enabled.  See http://openvpn.net/howto.html#mitm for more info.
Wed Dec 21 02:08:18 2016 NOTE: OpenVPN 2.1 requires '--script-security 2' or higher to call user-defined scripts or executables
Wed Dec 21 02:08:18 2016 LZO compression initialized
Wed Dec 21 02:08:18 2016 Control Channel MTU parms [ L:1542 D:138 EF:38 EB:0 ET:0 EL:0 ]
Wed Dec 21 02:08:18 2016 Socket Buffers: R=[163840->131072] S=[163840->131072]
Wed Dec 21 02:08:19 2016 RESOLVE: NOTE: usa.vanishedvpn.com resolves to 3 addresses
Wed Dec 21 02:08:19 2016 Data Channel MTU parms [ L:1542 D:1450 EF:42 EB:135 ET:0 EL:0 AF:3/1 ]
Wed Dec 21 02:08:19 2016 Local Options hash (VER=V4): '41690919'
Wed Dec 21 02:08:19 2016 Expected Remote Options hash (VER=V4): '530fdded'
Wed Dec 21 02:08:19 2016 UDPv4 link local: [undef]
Wed Dec 21 02:08:19 2016 UDPv4 link remote: [AF_INET]108.161.211.180:1194
Wed Dec 21 02:08:19 2016 TLS: Initial packet from [AF_INET]108.161.211.180:1194, sid=38377406 00d31e04
Wed Dec 21 02:08:19 2016 WARNING: this configuration may cache passwords in memory -- use the auth-nocache option to prevent this
Wed Dec 21 02:08:19 2016 VERIFY OK: depth=1, /CN=VanishedVPN
Wed Dec 21 02:08:19 2016 VERIFY OK: depth=0, /CN=138.68.92.51
Wed Dec 21 02:08:20 2016 Data Channel Encrypt: Cipher 'BF-CBC' initialized with 128 bit key
Wed Dec 21 02:08:20 2016 Data Channel Encrypt: Using 160 bit message hash 'SHA1' for HMAC authentication
Wed Dec 21 02:08:20 2016 Data Channel Decrypt: Cipher 'BF-CBC' initialized with 128 bit key
Wed Dec 21 02:08:20 2016 Data Channel Decrypt: Using 160 bit message hash 'SHA1' for HMAC authentication
Wed Dec 21 02:08:20 2016 Control Channel: TLSv1, cipher TLSv1/SSLv3 DHE-RSA-AES256-SHA, 2048 bit RSA
Wed Dec 21 02:08:20 2016 [138.68.92.51] Peer Connection Initiated with [AF_INET]108.161.211.180:1194
Wed Dec 21 02:08:22 2016 SENT CONTROL [138.68.92.51]: 'PUSH_REQUEST' (status=1)
Wed Dec 21 02:08:23 2016 PUSH: Received control message: 'PUSH_REPLY,dhcp-option DNS 8.8.8.8,dhcp-option DNS 8.8.4.4,redirect-gateway def1,route 192.168.255.1,topology net30,ping 10,ping-restart 60,ifconfig 192.168.255.58 192.168.255.57'
Wed Dec 21 02:08:23 2016 OPTIONS IMPORT: timers and/or timeouts modified
Wed Dec 21 02:08:23 2016 OPTIONS IMPORT: --ifconfig/up options modified
Wed Dec 21 02:08:23 2016 OPTIONS IMPORT: route options modified
Wed Dec 21 02:08:23 2016 OPTIONS IMPORT: --ip-win32 and/or --dhcp-option options modified
Wed Dec 21 02:08:23 2016 ROUTE default_gateway=192.168.1.1
Wed Dec 21 02:08:23 2016 TUN/TAP device tun0 opened
Wed Dec 21 02:08:23 2016 TUN/TAP TX queue length set to 100
Wed Dec 21 02:08:23 2016 do_ifconfig, tt->ipv6=0, tt->did_ifconfig_ipv6_setup=0
Wed Dec 21 02:08:23 2016 /sbin/ifconfig tun0 192.168.255.58 pointopoint 192.168.255.57 mtu 1500
Wed Dec 21 02:08:23 2016 /sbin/route add -net 108.161.211.180 netmask 255.255.255.255 gw 192.168.1.1
Wed Dec 21 02:08:23 2016 /sbin/route add -net 0.0.0.0 netmask 128.0.0.0 gw 192.168.255.57
Wed Dec 21 02:08:23 2016 /sbin/route add -net 128.0.0.0 netmask 128.0.0.0 gw 192.168.255.57
Wed Dec 21 02:08:23 2016 /sbin/route add -net 192.168.255.1 netmask 255.255.255.255 gw 192.168.255.57
Wed Dec 21 02:08:23 2016 Initialization Sequence Completed
```
Exit this with Ctrl+c

## Enable VPN at boot

```
crontab -e
```
Add the following line
```
@reboot openvpn --config /etc/openvpn/vanished.conf
```
## Setup Routing and NAT

Enable IP Forwarding:
```
echo -e '\n#Enable IP Routing\nnet.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```
Setup NAT fron the local LAN down the VPN tunnel:
```
sudo iptables -t nat -A POSTROUTING -o tun0 -j MASQUERADE
sudo iptables -A FORWARD -i tun0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o tun0 -j ACCEPT
```
Make the NAT rules persistent across reboot:
```
sudo apt-get install iptables-persistent
```
The installer will ask if you want to save current rules, select Yes

If you don't select yes, that's fine, you can save the rules later with sudo netfilter-persistent save

Make the rules apply at startup:
```
sudo systemctl enable netfilter-persistent
```
## VPN Kill switch
This will block outbound traffic from the Pi so that only the VPN and related services are allowed.

Once this is done, the only way the Pi can get to the internet is over the VPN.

This means if the VPN goes down, your traffic will just stop working, rather than end up routing over your regular internet connection where it could become visible.
```
sudo iptables -A OUTPUT -o tun0 -m comment --comment "vpn" -j ACCEPT
sudo iptables -A OUTPUT -o eth0 -p icmp -m comment --comment "icmp" -j ACCEPT
sudo iptables -A OUTPUT -d 192.168.1.0/24 -o eth0 -m comment --comment "lan" -j ACCEPT
sudo iptables -A OUTPUT -o eth0 -p udp -m udp --dport 1198 -m comment --comment "openvpn" -j ACCEPT
sudo iptables -A OUTPUT -o eth0 -p tcp -m tcp --sport 22 -m comment --comment "ssh" -j ACCEPT
sudo iptables -A OUTPUT -o eth0 -p udp -m udp --dport 123 -m comment --comment "ntp" -j ACCEPT
sudo iptables -A OUTPUT -o eth0 -p udp -m udp --dport 53 -m comment --comment "dns" -j ACCEPT
sudo iptables -A OUTPUT -o eth0 -p tcp -m tcp --dport 53 -m comment --comment "dns" -j ACCEPT
sudo iptables -A OUTPUT -o eth0 -j DROP
```
And save so they apply at reboot:
```
sudo netfilter-persistent save
```
If you find traffic on your other systems stops, then look on the Pi to see if the VPN is up or not.

You can check the status and logs of the VPN client with:
```
sudo systemctl status openvpn@Japan
sudo journalctl -u openvpn@Japan
```
## Configure Other Systems on the LAN

Now we're ready to tell other systems to send their traffic through the Raspberry Pi.

Configure other systems' network so they are like:

Default Gateway: Pi's static IP address (eg: `192.168.1.2`)
DNS: Something public like Google DNS (`8.8.8.8` and `8.8.4.4`)
Don't use your existing internet router (eg: `192.168.1.1`) as DNS, or your DNS queries will be visible to your ISP and hence may be visible to organizations who wish to see your internet traffic.

Optional: DNS on the Pi

To ensure all your DNS goes through the VPN, you could install dnsmasq on the Pi to accept DNS requests from the local LAN and forward requests to external DNS servers.
`
sudo apt-get install dnsmasq
`
You may now configure the other systems on the LAN to use the Pi (`192.168.1.2`) as their DNS server as well as their gateway.
