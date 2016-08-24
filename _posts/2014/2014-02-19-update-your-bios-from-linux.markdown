---
title: Update your BIOS from Linux
date: 2014-02-19 00:00:00 Z
tags:
- Linux
- BIOS
- DOS
---

I just struggled for quite some time to update my laptops BIOS (DELL Latitude). I am running only Linux (Ubuntu) as my main OS and didn't want to install Windows just for the sake of updating my BIOS.

I've found several helpful articles:

- [DellBIOS - from Ubuntu Wiki](https://wiki.ubuntu.com/DellBIOS)
- [ACHIEVEMENT UNLOCKED: BIOS UPGRADE WITHOUT USB OR CUSTOM TOOLS](http://blog.jasper.es/archives/19-Achievement-unlocked-bios-upgrade-without-usb-or-custom-tools.html)
- [How to make a bootable flash disk and to flash BIOS - from MSI]( http://www.msi.com/files/pdf/How_to_make_a_bootable_flash_disk_and_to_flash_BIOS_f.pdf)

... but none of them worked. However combining the knowledge from all those articles you can come up with a few easy steps how to do it.

## Few easy steps to update your BIOS
1. You'll need a USB Stick prepared.
I formatted mine with a 2GB FAT partition, using GParted and left the rest unassigned.
2. Download the new BIOS from your manufacturers page and extract it if necessary.
3. Download a, big enough, DOS-floppy image from [http://www.fdos.org/bootdisks/](http://www.fdos.org/bootdisks/) and extract it if necessary. 
I used the [10MB image](http://www.fdos.org/bootdisks/autogen/FDSTD10.zip), which is, at the time of this writing, in development, but worked for me.
4. Download [UNetbootin](http://unetbootin.sourceforge.net/) and make it executable. 
(`chmod +x unetbootin-linux-585`)
5. Run UNetbootin as root (`sudo ./unetbootin-linux-585`)
 - Select **Diskimage**, 
 - in the drop-down list choose **Floppy** instead of ISO and 
 - select the as a Diskimage the downloaded DOS Floppy (FDSTD10.IMG for me).
 - Choose the correct USB Drive to write to and press OK
 ![UNetbootin screenshot](/assets/2014/unetbootin_screenshot.png)
6. Before restarting, open the newly written USB Drive. You should see there the DOS files. Now copy the extracted BIOS, which you got from your manufacturers website, into the root directory.
7. Restart the computer and choose to boot from USB
8. Simply type in the name of the file (you can use `dir` to list all files as well as *tabulator* to autocomplete) and hit *enter*

The update should start now, just be careful if you don't have a QWERTY keyboard, since this is the layout, DOS and the updater use.