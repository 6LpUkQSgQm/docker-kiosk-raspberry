# simple-docker-kiosk-raspberry

### 1. Download Raspberry Pi OS (formerly Raspbian) Buster lite
<img src="https://www.raspberrypi.org/wp-content/uploads/2011/10/Raspi-PGB001.png" style="width:100px" />

Raspberry Pi OS Buster image that I downloaded from [Raspberry Pi OS](https://www.raspberrypi.org/downloads/raspberry-pi-os/)

### 2. Burn the Raspberry Pi OS image to the SD card
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/2/2d/Etcher-icon.png/220px-Etcher-icon.png" style="width:100px" />

To burn an image to the SD card you can use [Balena Etcher](https://www.balena.io/etcher/)

### 3. Enable ssh to allow remote login and WiFi network info

Open folder **ssh_wifi**, and add this two files in /Volumes/boot

### 4. Eject the micro SD card

Remove the mini-SD card from the adapter and plug it into the Raspberry Pi

### 5. Login remotely over WiFi

This part assumes that ssh is enabled for your image and that the default user is pi with a password of raspberry.

* Boot the Raspberry Pi and open up a terminal window
* Run the following commands:

```ssh-keygen -R raspberrypi.local```
```ssh pi@raspberrypi.local```


### 6. Get the latest updates

```sudo apt-get update -y```
```sudo apt-get upgrade -y```

