# UsingRaspberryPi
TL;DR Configuration
## Backup and edit config HAOS
Connect a USB keyboard and an HDMI monitor (the Pi 4 uses micro-HDMI) and press Enter. You’ll see the ha > prompt.
Open the root shell: type "login". The prompt changes to #.

Remote connect SSH to the host (port 22222)
This is the same as method first, but over the network. It needs one-time setup: you put your SSH public key in a file named authorized_keys on a USB stick named CONFIG, then import it with ha os import. The official guide is “Debugging the Home Assistant Operating System”. After that, connect with:
ssh root@homeassistant.local -p 22222
Back up the file first: cp /mnt/boot/config.txt /mnt/boot/config.txt.bak
Edit the file: vi /mnt/boot/config.txt (HAOS mounts the boot partition there). Add the two lines, then save with Esc :wq.
Reboot: reboot
## Config devices
RS485 CAN HAT (B) 
> dtparam=spi=on
dtoverlay=mcp2515-can0,oscillator=16000000,interrupt=25,spimaxfrequency=1000000
dtoverlay=sc16is752-spi1,int_pin=24
Checking
> dmesg | grep -i -E 'sc16is7|mcp251|spi'
ls /dev/ttySC*
You should see /dev/ttySC0 and /dev/ttySC1,
The mcp2515-can0 line is only for the CAN port. Home Assistant has no built-in CAN support, and on HAOS you’d have to bring can0 up from the host shell yourself.
DS3231 HW-022
> dtparam=i2c_arm=on
dtoverlay=i2c-rtc,ds3231
Checking
> dmesg | grep -i rtc
timedatectl
