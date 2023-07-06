From https://github.com/B83C/max2140_bin/issues/1

# Preface
So, I was trying to stop router advertisements from coming from the router as I was using it as a dumb access point. I only see option to disable DHCPv6 in the web interface. Interestingly, uncommenting some html tags in the IPv6 tab presents option to enable or disable Router Advertisement, but that doesn't work, as I could see RAs in Wireshark. 

In the end, I went into the shell and escaped from it. After exploring the contents of rootfs, I see radvd.conf in /var, but that is only temporary, as the file is recreated on every boot. I noticed only a single instance of radvd can be run, so I thought about running radvd ahead of time via /etc/rc3.d before it could get started by 'sdm' process. I guess I screwed up as the router could no longer boot!

The steps below will recover the router from brick if you have done something in rootfs to make it unbootable.

# Item needed
To debrick the router, you'll need a USB to TTL adapter.

# Serial port
First, we need access to the bootloader, hence, we need to connect to the serial port. See the following images to guide you. Use a serial console software such as PuTTY. Connect at a baud rate of 115200.

<img src="https://github.com/B83C/max2140_bin/assets/58171737/7a06fc47-ee0c-4540-a85d-30b894929c15" width="500em">
<img src="https://github.com/B83C/max2140_bin/assets/58171737/045ad65d-57e6-491a-b606-58da4f9183e0" width="500em">
<img src="https://github.com/B83C/max2140_bin/assets/58171737/c7e5c5f8-0eb3-4271-9056-bb36bd0d4b2d" width="500em">

# Switching boot image
I found out that the router contains two boot image -- latest and previous. I assume when the router is updated it writes to the secondary boot partition. We can switch to the secondary boot image in the CFE bootloader. When connected via serial, restart the router, and interrupt the boot process by spamming keys in the serial console. You should be greeted by the following shell:

![image](https://github.com/B83C/max2140_bin/assets/58171737/6583af55-39bf-4d34-820a-1d7b7cfc8834)

We are interested in the `c` command, which changes boot line parameters. Press enter until the boot image line. Here, instead of 0, enter 1. After finishing with the command, continue with the boot process with the `r` command.

![image](https://github.com/B83C/max2140_bin/assets/58171737/5333d7c7-bab3-46fa-b554-a95cf5dbc610)

The bootloader simply swaps memory address of boot images. From here on out, your router should boot normally.

<details><summary>Addendum</summary>
<img src="https://github.com/B83C/max2140_bin/assets/58171737/e042334b-e498-42bb-a9ec-e17863d89fdf"/>
<p>Boot image: latest</p>
<img src="https://github.com/B83C/max2140_bin/assets/58171737/e1545208-7068-4f16-af57-52d44385e261"/>
<p>Boot image: previous</p>
<img src="https://github.com/B83C/max2140_bin/assets/58171737/16b5554e-a184-41fb-86a7-09b20938ce30"/>
<p>Boot image: latest</p>
<img src="https://github.com/B83C/max2140_bin/assets/58171737/d1d6e4fd-374f-40fd-91c6-e6ea7e4d2de2"/>
<p>Boot image: previous</p>
</details>

# Restoring original boot image
Now having only a single boot image doesn't feel good as bricking that one would get you in a tough pickle. To restore it, it is a simple as duplicating current boot image to where your original boot image resides. To do this, we will duplicate partitions from current UBI boot image into `/tmp`, and then update the original UBI boot image partitions with the partitions you just saved.

From here on, you'll need to escape the restricted shell. You can use this repo's way to obtain root access, or, escaping the restricted shell another way:
- Login to restricted shell using MaxARSysOpr:!!bestHome!!XXXX through serial console.
- Enter: `echo $(sh)`
- Then: `exec >&2`
- Finally: `sh`

You'll then find yourself in an unrestricted shell. However, it is only temporary, as the restricted shell imposes time limit on executing commands. Periodically exit into restricted shell from time to time as the console can bug out. If it bugs out, CTRL-D until you log back out from shell, then log back in, and do the same procedure.

See current mtd partitions with `mtdinfo -a`. We are interested in `image` and `image_upgrade`, where the prior is current boot image and the latter is the broken one. They are `/dev/mtd5` and `/dev/mtd4` respectively. Verify the following image:

![image](https://github.com/B83C/max2140_bin/assets/58171737/67da5c29-d222-44f6-923f-7f810a482521)

As `/dev/mtd4` and `/dev_mtd5` are UBI devices, we have to mount them using `ubiattach` command.
```
ubiattach -p /dev/mtd4 -d 2
ubiattach -p /dev/mtd5 -d 3
```

There are four UBI volumes in `ubi2` and `ubi3`.
```
/dev/ubiX_0 rootfs_ubifs
/dev/ubiX_1 METADATA
/dev/ubiX_2 METADATA_COPY
/dev/ubiX_10 filestruct_full.bin
```

We will copy them to `/tmp`.
```
dd if=/dev/ubi3_0 of=/tmp/ubi3_0.ubi
dd if=/dev/ubi3_1 of=/tmp/ubi3_1.ubi
dd if=/dev/ubi3_2 of=/tmp/ubi3_2.ubi
dd if=/dev/ubi3_10 of=/tmp/ubi3_10.ubi
```

Then, write them to `ubi3` -- do not use `dd`.
```
ubiupdatevol /dev/ubi2_0 /tmp/ubi3_0.ubi
ubiupdatevol /dev/ubi2_1 /tmp/ubi3_1.ubi 
ubiupdatevol /dev/ubi2_2 /tmp/ubi3_2.ubi 
ubiupdatevol /dev/ubi2_10 /tmp/ubi3_10.ubi 
```

At this point, we are done. Do not forget to unmount. Or not.
```
ubidetach -p /dev/mtd4
ubidetach -p /dev/mtd5
```

Reboot the router and change boot image config in the bootloader back to latest. Your original boot image partition will now be bootable.
