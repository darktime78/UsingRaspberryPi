# UsingRaspberryPi
TL;DR Configuration
## Eeprom
Get version
> __ha> os boards raspberrypi firmware__     
__rpi-eeprom-update__     
__/opt/vc/bin/vcgencmd bootloader_version__

Update      
> __ha> os boards raspberrypi firmware update__      
__sudo rpi-eeprom-update -d -a__      
         
## Backup and edit config HAOS
Connect a USB keyboard and an HDMI monitor (the Pi 4 uses micro-HDMI) and press Enter. You’ll see the __ha >__ prompt.
> Open the root shell: type __login__. The prompt changes to __#__.

Remote connect SSH to the host (port 22222)
This is the same as method first, but over the network. It needs one-time setup: you put your SSH public key in a file named authorized_keys on a USB stick named CONFIG, then import it with ha os import. The official guide is “Debugging the Home Assistant Operating System”. After that, connect with:
> __ssh root@homeassistant.local -p 22222__

Back up the file first:
> __cp /mnt/boot/config.txt /mnt/boot/config.txt.bak__

Edit the file:
> __vi /mnt/boot/config.txt__ (HAOS mounts the boot partition there). Add the two lines, then save with __Esc :wq__.

Reboot:
> __reboot__

## Config devices
RS485 CAN HAT (B) 
> __dtparam=spi=on__     
__dtoverlay=mcp2515-can0,oscillator=16000000,interrupt=25,spimaxfrequency=1000000__     
__dtoverlay=sc16is752-spi1,int_pin=24__     

Checking
> __dmesg | grep -i -E 'sc16is7|mcp251|spi'__    
__ls /dev/ttySC*__    
You should see __/dev/ttySC0__ and __/dev/ttySC1__,
The __mcp2515-can0__ line is only for the CAN port. Home Assistant has no built-in CAN support, and on HAOS you’d have to bring __can0__ up from the host shell yourself.

DS3231 HW-022
> __dtparam=i2c_arm=on__     
__dtoverlay=i2c-rtc,ds3231__     

Checking
> __dmesg | grep -i rtc__     
You should show __rtc-ds1307 … registered as rtc0__
__timedatectl__     

## Some USB-SATA adapters don’t work properly with the Pi 4’s fast USB mode (called UAS).
Typical symptoms are boot hangs, random disconnects, or very slow disk speed. Many cheap mSATA adapters use the affected chips (JMicron, some ASMedia). Two fixes:     
Quick test: plug the SSD into a black USB 2 port. If it boots there, the adapter is the problem.     
Permanent fix: turn __UAS__ off for that adapter.     
Find its USB ID:     
On Windows: Device Manager → the drive’s adapter → Properties → Details → Hardware IDs. __VID_152D&PID_0578__ means 152d:0578.     
On Linux: run __lsusb__.     
Open __cmdline.txt__ on __hassos-boot_. It’s a single line. Add this to the beginning, followed by a space, keeping everything on one line (use your adapter’s ID):     
> __usb-storage.quirks=152d:0578:u__

## Detect usb hard driver adapter chipset and option     
> __grep -H . /sys/bus/usb/devices/*/product__      
> __sudo smartctl -i /dev/sda__
> __lsscsi__            
> __lsblk --discard__
if DISC-GRAN and DISC-MAX is zero, TRIM disable.

The ASM225 seamlessly translates the Linux UNMAP command into an ATA TRIM command for the drive. This is critically important for SSDs: without TRIM, the SSD will begin to degrade in performance after just a few months of active use. 
Enable TRIM for USB.     
> __sudo nano /etc/udev/rules.d/10-trim.rules__     
__ACTION=="add|change", ATTRS{idVendor}=="174c", ATTRS{idProduct}=="55aa", ATTR{queue/discard_max_bytes}="33554432"__      
__sudo reboot__    
__sudo fstrim -v /__     

