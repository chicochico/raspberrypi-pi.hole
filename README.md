# Pi hole

## Router configurations
After setting up pi.hole in a rasberry pi, set the following settings in the router.

#### Always assign same IP address
```
WLAN > Funknetz > "Bekkante WLAN-Gerate im Funknetz"

# Find the pi.hole and click on the pencil on the right side.

---

Addressen im Heimnetz (IP-Adressen) 
    IPv4-Adresse
        [x] Diesem Netzwerkgerat immer die gleiche IPv4-Adresse zuweisen.
```



#### DNS server
This is the dns server used by default by devices connected to the router.

```
Internet > Zugangsdaten > [DNS-Server]

---

# Tick the option
[x] Andere DNSv4-Server verwenden
    # Use the fixed ip assigned to raspberry pi running pi.hole: `194.168.178.29`
```


#### Set Pi Hole as the default DNS offered by DHCP
```
Heimnetz > Netzwerk > [Netzferkeinstellungen]

IP-Adressen
    IPv4-Eientellungen

---

Heimnetz
    [x] DHCP-Server aktivieren
        Lokaler DNS-Server: 194.168.178.29 # fixed ip assigned to pi.hole
```

## Preparing SD card with rasbpian
On MacOS. Download Raspberry Pi OS Lite image from [here](https://www.raspberrypi.com/software/operating-systems/).
```
# Checksum check
> shasum -a 256 ~/Downloads/2023-05-03-raspios-bullseye-armhf-lite.img.xz
b5e3a1d984a7eaa402a6e078d707b506b962f6804d331dcc0daa61debae3a19a  /Users/fchiang/Downloads/2023-05-03-raspios-bullseye-armhf-lite.img.xz
# Make sure the hash match with the hash on the website

# List volumes
> diskutil list
/dev/disk4 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *31.1 GB    disk4
   1:                 DOS_FAT_32 BOOT                    31.1 GB    disk4s1

# Identify the SD card volume and note the identifier, for ex: /dev/disk4
# Format FAT32 with a partition named BOOT
> diskutil eraseDisk FAT32 BOOT MBRFormat /dev/disk4

# Unmount disk for copying the image
> diskutil unmountDisk /dev/disk4

# Copy downloaded img
# Make sure to decompress beforehand if archive is compressed
> sudo dd if=<PATH-TO-IMG> of=/dev/disk4 conv=sync
3842048+0 records in
3842048+0 records out
1967128576 bytes (2.0 GB, 1.8 GiB) copied, 497.883 s, 4.0 MB/s
```

Configure headless wifi connection, create a file in the sd card root directory named `wpa_supplicant.conf` with the following contents:
```
 country=DE
 ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
 update_config=1
 
 network={
     ssid="<WIFI-NETWORK-SSID>"
     psk="<PASSWORD>"
 }
```

To enable SSH, create an empty file named `ssh` in the volume root, and add an additional file named `authorized_keys` with your public key.

Unmount disk:
```
> diskutil unmountDisk /dev/disk4
```

Create OS user (user `pi` is not created by default any longer). Add a file named `userconf.txt` with the content:
```
<USERNAME>:<HASHED-PASSWORD>
```

Generating hashed password:
```
# -6 specifies the hash algorithm to be used, here the SHA-512-based Unix crypt format will be used.
echo "mypassword" | openssl passwd -6 -stdin
```
**NOTE**: Default openssl from mac does not have the option `-6`.

Add your public key for ssh without password:
```
ssh-copy-id -i ~/.ssh/id_ed25519.pub pi@<PI-IP-ADDRESS>
```


## Installing pi.hole
Run the command and follow the onscreen wizard:
```
curl -sSL https://install.pi-hole.net | bash
```
