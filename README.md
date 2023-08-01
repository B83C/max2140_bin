## max2140_bin

# Enabling ssh/telnet/ftp server
The key to full access manifests itself in the form of !!bestHOME!!xxxx where xxxx is shy last 4-digit of the serial number. You must disguise yourself as MaxARSysOpr.

Listen to me, hacker, your time has come, the clue to breaking free is in the access control section of the Security panel. Now, go ahead!

# Gaining root access for custom installations 
Prerequisites : 
- Working MaxARSysOpr password
- A modicum of knowledge 
- Enabled telnetd/ssh from the UI@192.168.1.254

Steps:

1. Log in to either telnet, shh or serial port.
2. Run `echo $(sh)`
3. Followed by `wget --no-check-certificate https://github.com/B83C/max2140_bin/blob/main/busybox?raw=true -O /tmp/busybox && chmod +x /tmp/busybox && /tmp/busybox telnetd -l /bin/sh -p 1234` 
6. You're basically done! Connect to the router with `telnet 192.168.1.254 1234` (if you are on *nix). 
7. Voila full root access to the router! Be warned that you have just added another vulnerability to the system (that won't persist across reboot, of course), and I am not responsible for any damages caused.

# Decrypting/Encrypting the configuration file
I was tired of using the openssl client, so I wrote a utility in Rust. Check it out (here)[decipher]
To use the tool, you can either run `./decipher <input_file> <output_file>`, or let it prompt you for the correspending paths by not specifying the files.
FYI: Configuration related operations are accessible under Management > Configuration

# Files

| Path           | Uses                                                                                  |
|----------------|---------------------------------------------------------------------------------------|
| busybox        | Pre-built busybox binary                                                              |
| busybox_config | Build config file, copy it into the root path of the busybox repo as .config to build |
| scripts/rrmnt  | Convenient helper tool to remount / of the router as rw/ro                            |
| decipher/*     | Helper tools written in Rust to help decipher the configuration file (currently closed source) |

# Hurdles
- No firmware available as of now, if you have a stock router unmodified, kindly let me know cuz I want a working back up for my router.
- Some software problems that came along with the stock firmware

# Known problems that can be fixed / worked around
- Most importantly, dhcpcd static ip, the web UI is known not to work properly, but by gaining root access, you can fix this yourself by editing /etc/udhcpd.conf (remember to remount the rootfs as rw)
- And lack of features, i.e. custom dyn ip server, VPN (it has ipsec), and some other feasible stuff... Btw I got v2ray to run on this thing but the only problem is that maxis is very ungenerous, they limit your upload speed to 50mbps as opposed to time where upload and download are the same! 
- *Devices disconnect immediately upon authenticating (especially old devices)* : need to change ieee80211w=1 to ieee80211w=0 in /tmp/wl1_hapd.conf and restarting the interface wl1. It can be fixed by doing ```nvram set wl0_mfp=0``` ```nvram set wl0.1_mfp=0``` ```nvram set wl1_mfp=0``` ```nvram set wl1.1_mfp=0``` followed by ```nvram commit```( This will turn off Management Frame Protection in favor of backward compatibility with older devices? Not sure, but it allows android 5/6 devices to connect without any issues)
  
# Legal?
Well, Idk, the goal here is to provide end-users with the rights to add their custom binaries to the stock router, extending its functionalities ( and also to reduce e-waste, since we do not have to get a new router for what we demand ). The benefits are endless, that said, I am not responsible for any damages done to your router, and you are supposed to have at least some knowledge about embedded linux / linux. Happy modding! 
