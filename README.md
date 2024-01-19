
# openIPC-GK7205V300
## openIPC GK7205V300 install and setup

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

### Step 3. Connect to UART port of your camera.
Install minicom terminal:
```bash
sudo apt install minicom
```

Connect you serial-to-USB adapter cable to the computer and the wires to the camera serial interface. **Make sure the voltage is set to 3.3V** .



Start a sessions with:

```bash
minicom -b 115200 -8 --capturefile=ipcam-$(date +%s).log --color=on -D /dev/ttyUSB0
```
Power the camera with its standard power adapter. You will see the booting log in your terminal window.

(Use `Ctrl-a` followed by `x` to exit the session.)



