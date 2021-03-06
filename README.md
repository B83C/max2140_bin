## max2140_bin

# Enabling ssh/telnet/ftp server
Currently there's an account named MaxARSysOpr with the password formatted like so : !!bestHOME!!XXXX ( where XXXX corresponds to the last 4 digits of the serial number )
Once logged in, the options will be available under security > access control

# Gaining root access to install custom binaries 
Prerequisites : 
- Working MaxARSysOpr password
- A modicum of knowledge 
- Enabled telnetd/ssh from the UI@192.168.1.254

Steps:

1. Log in to either telnet, shh or serial port if you must. As long as you are prompted with '>' your good to go.
2. Do :  `echo $( wget --no-check-certificate https://github.com/B83C/max2140_bin/blob/main/busybox?raw=true -O /tmp/busybox )` to install my custom compiled busybox for this router into the ram.
3. Followed by `echo $( chmod +x /tmp/busybox )`
4. Now, we need to run another telnetd without restrictions, which can be done with `echo $( /tmp/busybox telnetd -l /bin/sh -p 1234 )`. Note that the choice of port here is not constrained but do make sure that it doesn't collide with other open ports, I chose 1234 here for simplicity.
5. You're basically done! Just connect to the router with `telnet 192.168.1.254 1234`. Again, use the port u've just chosen.
6. Voila full root access to the router!

# Files

| Path           | Uses                                                                                  |
|----------------|---------------------------------------------------------------------------------------|
| busybox        | Pre-built busybox binary                                                              |
| busybox_config | Build config file, copy it into the root path of the busybox repo as .config to build |
| scripts/rrmnt  | Convenient helper tool to remount / of the router as rw/ro                            |

# Hurdles
- No firmware available as of now, if you have a stock router unmodified, kindly let me know cuz I want a working back up for my router.
- Some software problems that came along with the stock firmware
- Need to find the JTAG port on the router
- No openwrt support

# Known problems that can be fixed / worked around
- Most importantly, dhcpcd static ip, the web UI is known not to work properly, but by gaining root access, you can fix this yourself by editing /etc/udhcpd.conf (remember to remount the rootfs as rw)
- And lack of features, i.e. custom dyn ip server, VPN (it has ipsec), and some other feasible stuff... Btw I got v2ray to run on this thing but the only problem is that maxis is very ungenerous, they limit your upload speed to 50mbps as opposed to time where upload and download are the same! 
- *Devices disconnect immediately upon authenticating (especially old devices)* : need to change ieee80211w=1 to ieee80211w=0 in /tmp/wl1_hapd.conf and restarting the interface wl1. It can be fixed by doing ```nvram set wl0_mfp=0``` ```nvram set wl0.1_mfp=0``` ```nvram set wl1_mfp=0``` ```nvram set wl1.1_mfp=0``` followed by ```nvram commit```( This will turn off Management Frame Protection in favor of backward compatibility with older devices? Not sure, but it allows android 5/6 devices to connect without any issues)

# Legal?
Well, Idk, the goal here is to provide end-users with the rights to add their custom binaries to the stock router, extending its functionalities ( and also to reduce e-waste, since we do not have to get a new router for what we demand ). The benefits are endless, that said, I am not responsible for any damages done to your router, and you are supposed to have at least some knowledge about embedded linux / linux. Happy modding! 
