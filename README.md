

# OpenIPC-GK7205V300
## OpenIPC GK7205V300 install and setup

==**Note:** this guide is made using Linux Mint 21.2 . If you are using another distribution, probably you have to change the commands accordingly and the output may be different.==


###  Step 1. Prepare your camera
You will need to find where are the serial TX and RX of the camera, and GND. Some cameras may have connector, other may need soldering wires on the correct pads of the camera board.
You have to provide power via the included Ethernet cable and suitable power supply with output 12V (some people have reported 5V may be enough if the camera does not have LEDs)

###  Step 2. Install TFTP server tftpd-hpa

Update the system and install the server:
```bash
sudo apt update
sudo apt install tftpd-hpa
```
After the installation is complete, verify the server is running and has been successfully installed. Run this:
```bash
sudo systemctl status tftpd-hpa.service
```
And the result should include something like this:
```
● tftpd-hpa.service - LSB: HPA's tftp server
     Loaded: loaded (/etc/init.d/tftpd-hpa; generated)
     Active: active (running) since Fri 2024-01-19 21:58:02 EET; 3min 49s ago
       Docs: man:systemd-sysv-generator(8)
    Process: 16408 ExecStart=/etc/init.d/tftpd-hpa start (code=exited, status=0/SUCCESS)
      Tasks: 1 (limit: 16318)
     Memory: 1.1M
        CPU: 6ms
     CGroup: /system.slice/tftpd-hpa.service
             └─16416 /usr/sbin/in.tftpd --listen --user tftp --address :69 --secure /srv/tftp
```

If the service is in a dead state, you have to activate it:

```bash
sudo systemctl start tftpd-hpa
```

You can enable the server to automatically start at boot time (optional):

```bash
sudo systemctl enable tftpd-hpa
```

To setup and restart the service run:
```bash
sudo sed -i '/^TFTP_OPTIONS/s/"$/ --create"/' /etc/default/tftpd-hpa
sudo systemctl restart tftpd-hpa.service
```
this will create configuration file pointing to the server directory `/srv/tftp`

Set the necessary permissions on this directory:
```
sudo chmod -R 777 /srv/tftp
sudo chown -R nobody:nogroup /srv/tftp
```

Test if the TFTP server is working as expected. On other device connected to your network run:

```bash
tftp 192.168.0.112 
```
In the above command use the IP address of the machine where you installed the TFTP server.
If the connection is successful, you will see a _tftp_ prompt:

```bash
tftp>
```
Type _`quit`_ to exit the prompt.

### Step 3. Connect to UART port of your camera
Install minicom terminal:
```bash
sudo apt install minicom
```

Connect you serial-to-USB adapter cable to the computer and the wires to the camera serial interface. **Make sure the voltage is set to 3.3V** .

Check the name of your serial-to-USB device:
```
ls /dev/ttyUSB*
```
Probably the output will be `/dev/ttyUSB0` You will need it for the next command.

Start a sessions with:

```bash
minicom -b 115200 -8 --capturefile=ipcam-$(date +%s).log --color=on -D /dev/ttyUSB0
```
Power the camera with its standard power adapter. You will see the booting log in your terminal window.

(Use `Ctrl-a` followed by `x` to exit the session if you have to.)

### Step 4. Get access to the bootloader

Reboot the camera and interrupt its boot sequence to access bootloader console by pressing a key combination, between the time the bootloader starts and before Linux kernel kicks in. Key combinations differ from vendor to vendor but, in most cases, it is `Ctrl-C`,  `Enter`, `Esc`, `*` or just any key.
On succeeded you will get a command prompt.

Bootloader may be locked and not giving prompt even when pushing the buttons.

#### Access the bootloader prompt by temporary bootloader flashed with the `burn` utility


https://www.youtube.com/watch?v=XsqKEYvpwyU

1. Download Burn utility from the site https://github.com/OpenIPC/burn
2. Extract the archive and rename the folder to `burn`
3. Download bootloader from here: **[Archive of current bootloaders for devices](https://github.com/OpenIPC/firmware/releases/tag/latest)** and save it in the `burn` folder
4. open a terminal in the same folder and run the command
```
./burn --chip gk7205v300 --file=u-boot-gk7205v300-universal.bin --break; screen -L /dev/ttyUSB0 115200
```
5.  power the camera
6. wait for the camera to accept the new temporary bootloader 
7. when it starts booting the OpenIPC U-boot bootloader , press any key to get to the U-boot console
8. The temporary bootloader has tftp and when the console is accessed you can continue with the automatic generated instructions for  **Save the original firmware** and **Flash full OpenIPC Firmware image**

#### Unlock flash on gk7205v300
I don't know what exactly are doing the commands after the burn. There is no explanation in the wiki.
```
$ ./burn --chip gk7205v300 --file=u-boot-gk7205v300-universal.bin --break; screen -L /dev/ttyUSB0 115200
goke # sf probe
@do_spi_flash_probe() flash->erase_size: 65536, flash->sector_size: 0
goke # sf lock 0
unlock all block.
```



### Step 5. Determine the flash memory size

Most IP cameras are equipped with 8 or 16 MB NOR or NAND flash memory. You can check the type and size of the chip installed on of your camera in the bootloader log output. You'll see something like this:
```
U-Boot 2010.06-svn (Oct 21 2016 - 11:21:29)

Check Flash Memory Controller v100 ... Found
SPI Nor(cs 0) ID: 0xс2 0x20 0x18
spi_general_qe_enable(294): Error: Disable Quad failed! reg: 0x2
Block:64KB Chip:16MB Name:"MX25L128XX"
SPI Nor total size: 16MB
```
Another example:
```
U-Boot 2013.07 (Feb 27 2019 - 02:05:08)

DRAM:  64 MiB
MMC:   msc: 0
SF: Detected EN25QH64
```
Which shows the flash memory model (`EN25QH64`) that you can look up online to find a data sheet. Also, `64` in the model number hints for a 64 Megabits memory, which is equivalent to 8MB. Similarly, `128` would be equivalent to 16MB.

This is what I have:
```
System startup

Uncompress Ok!

U-Boot 2016.11-g4bd9c94-dirty (Apr 22 2022 - 21:25:43 +0800)gk7205v300

Relocation Offset is: 0771f000
Relocating to 47f1f000, new gd at 47edeef0, sp at 47edeed0
SPI Nor:  Check Flash Memory Controller v100 ... Found
SPI Nor ID Table Version 1.0
@hifmc_spi_nor_probe(), SPI Nor(cs 0) ID: 0xa1 0x40 0x18 <Read>
@hifmc_spi_nor_probe(), SPI Nor(cs 0) ID: 0xa1 0x40 0x18 <Found>
SPI Nor(cs 0) ID: 0xa1 0x40 0x18
eFlashType: 30.
@XmSpiNor_initSr3() need No SR3!
@XmSpiNor_printWps(), WPS Enabled!
Flash Name: XM_FM25Q128A{0xA14018), 0x1000000.
@hifmc_spi_nor_probe(), XmSpiNor_ProtMgr_probe(): OK.
@XmSpiNor_enableQuadMode(), Quad was Disabled, SRx: [2, 0x8].
Block:64KB Chip:16MB Name:"XM_FM25Q128A"
CONFIG_CLOSE_SPI_8PIN_4IO = y.
read->iftype[0: STD, 1: DUAL, 2: DIO, 3: QUAD, 4: QIO]: 1.
lk[7 => 0x1000000], Total Lock Blks: 286 <WPS=1>
SRx val: {[1, 0x20], [1, 0x8], [1, 0x0], [0, 0x0]}, SrVal: 0x700000000000820.
SPI Nor total size: 16MB
In:    serial
Out:   serial
Err:   serial
Net:   eth0
autoboot:  0 
@do_spi_flash_probe() flash->erase_size: 65536, flash->sector_size: 0
device 0 offset 0x40000, size 0x540000

SF: 5505024 bytes @ 0x40000 Read: OK
srcAddr: 0x43000000, dstAddr: 0x42000000, filename: boot/uImage.
created_inode 0x47edfc18
find_squashfs_file: name bin, start_block 0, offset 1824, type 1
find_squashfs_file: name boot, start_block 0, offset 1996, type 1
read inode: name boot, sb 0, of 1996, type 1
find_squashfs_file: name uImage, start_block 0, offset 1856, type 2
read inode: name uImage, sb 0, of 1856, type 2
write_file: regular file, blocks 27
len 1723581
### get_squashfs_file OK: loade 1723581 bytes to 0x42000000
## Booting kernel from Legacy Image at 42000000 ...
   Image Name:   Linux-4.9.37
   Image Type:   ARM Linux Kernel Image (uncompressed)
   Data Size:    1723517 Bytes = 1.6 MiB
   Load Address: 40008000
   Entry Point:  40008000
   Loading Kernel Image ... OK
using: ATAGS
at setup_xminfo_tag() g_nXmBootSysIndex: 0, g_nXmRomfsIndex: 0.

Starting kernel ...

```

### Step 6. Save the original firmware

After you get access to the bootloader console, run `help` to get a list of available commands. Check if you have `tftp` among them. If you do, then saving the original firmware should be easy.

If your bootloader does not have `tftp`, you can still make a copy of the original firmware. [Read here for more](https://github.com/OpenIPC/wiki/blob/master/en/help-uboot.md).

Check the system environment using `printenv` command. Look for the parameters:

| Parameter   | Description |
| :----------- | :----------- |
| `ipaddr`    | camera IP address |
| `netmask`   | netmask of your camera|
| `gatewayip` |	IP address of the network gateway|
| `serverip`  | IP address of your TFTP server|


Assign the values by `setenv` command (use IP addresses and netmask corresponding to your local network), then save the new values into environment with `saveenv` command.

```
setenv ipaddr 192.168.1.10
setenv netmask 255.255.255.0
setenv gatewayip 192.168.1.1
setenv serverip 192.168.1.1
saveenv
```
To dump the original firmware, you need to save the contents of camera's flash memory to a file. For that, you must first load the contents into RAM: 
1. Initialize the Flash memory. 
2. Clean a region of RAM large enough to fit whole content of flash memory chip. 
3. Read contents of the flash from into that region
4. Export it to a file on the TFTP server

Please note, that flash type, size and starting address differ for different cameras! For exact commands please use [automatically generated instructions](https://openipc.org/supported-hardware/) for your hardware.

### Step 7. Install OpenIPC firmware

#### Preparing the firmware and the TFTP server

Go to [https://openipc.org/supported-hardware](https://openipc.org/supported-hardware), find your SoC in the table of supported hardware. Download the pre-compiled firmware file for your processor onto your PC.

The TFTP server is serving files from `/srv/tftp` directory. Extract files from the bundle you just downloaded into that directory.
```bash
sudo tar -C /srv/tftp/ -xvf openipc.*.tgz
```
#### Preparing the camera for flashing

Connect to the camera via the UART port and access the bootloader console. 
Follow the commands in  [automatically generated instructions](https://openipc.org/supported-hardware/) for your hardware.
There you have to find your hardware and select the link **Generate an installation guide**.
Fill the values in the fields according to your setup:
1. generate MAC address
2. Camera IP address
3. TFTP server IP address
4. select Type and size of flash memory chip (only NOR 8M is available for the FPV version of OpenIPC)
5. Firmware version: FPV
6. Network interface 
7. SD card slot 

and click on `Generate instructions guide`
You will see commands for:
1. Save the original firmware (already done)
2. Flash full OpenIPC Firmware image (continue with these commands)

The commands will do the following: 

Set the component parameters to the appropriate environment variables. Set environment variables for loading the Linux kernel and the root file system of the new firmware. Set environment variables for the camera to access local network, where:

 `ethaddr` is the original camera MAC address, 
 `ipaddr` is camera's IP address on the network, 
 `gatewayip` is the IP address of a router to access the network, 
 `netmask` is the subnet mask, 
 `serverip` is am IP address of the TFTP server 
 
 Save updated values to flash memory.



#### Installation

For exact commands please use [automatically generated instructions](https://openipc.org/supported-hardware/) for your hardware.

==**Note:** Enter commands line by line! Do not copy and paste multiple lines at once!
Pay attention to the messages on the terminal screen! If any of the commands throws an error, find out what went wrong. Maybe you made a typo? In any case, do not continue the procedure until all previous commands succeed. Otherwise, you might end up with a bricked camera!==

### Step 8. First boot

If all previous steps are done correctly, your camera should start with the new firmware. Welcome to OpenIPC!

After the first boot with the new firmware you need to clean the overlay partition. Run this in your terminal window:
```
firstboot
```

### Flash bootloader with CH341A programmer

Update system and isntall `flashrom`. This command is for Linux Mint/Ubuntu/Debian for other distributions search the internet how to install `flashrom`.

```
sudo apt update
sudo apt install flashrom 
```
or install from source
First install dependencies:
```
sudo apt-get install git -y \
    build-essential -y \
    libpci-dev -y \
    libusb-dev -y \
    libusb-1.0-0-dev -y \
    libftdi-dev -y
```
clone the repository
```
git clone https://review.coreboot.org/flashrom.git
```
Enter to the downloaded directory and build flashrom.
```
cd flashrom
make
```
And install
```
make install
```
In my case I had permission denied error so did it with `sudo`
```
sodo make install
```

1. Connect the clip to the BIOS chip, nothing should be powered
2. Connect the clip or adapters to the CH341a programmer
3. Connect the CH341a programmer to USB
4. Read the data from the chip and backup (may backup twice and compare the check sums of the two backups)
```
sudo flashrom --programmer ch341a_spi -r backup1.bin
```
5. Write the new file to the chip
```
sudo flashrom --programmer ch341a_spi -w u-boot-gk7205v300-universal.bin
```


