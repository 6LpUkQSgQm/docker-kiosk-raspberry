# simple-docker-kiosk-raspberry

## Kiosk mode

### 1. Download Raspberry Pi OS (formerly Raspbian) Buster lite

<img src="https://www.raspberrypi.org/wp-content/uploads/2011/10/Raspi-PGB001.png" width="100px"/>

Raspberry Pi OS Buster image that I downloaded from [Raspberry Pi OS](https://www.raspberrypi.org/downloads/raspberry-pi-os/)

### 2. Burn the Raspberry Pi OS image to the SD card

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/2/2d/Etcher-icon.png/220px-Etcher-icon.png" width="100px"/>

To burn an image to the SD card you can use [Balena Etcher](https://www.balena.io/etcher/)

### 3. Enable ssh to allow remote login and WiFi network info

Open folder **ssh_wifi**, and add this two files in /Volumes/boot

### 4. Eject the micro SD card

Remove the mini-SD card from the adapter and plug it into the Raspberry Pi

### 5. Login remotely over WiFi

This part assumes that ssh is enabled for your image and that the default user is pi with a password of raspberry.

* Boot the Raspberry Pi and open up a terminal window
* Run the following commands:

```shell
ssh-keygen -R raspberrypi.local
ssh pi@raspberrypi.local
```

### 6. Get the latest updates

```shell
sudo apt-get update -y
sudo apt-get upgrade -y
```

### 7. Install minimum GUI components

Before you can run the Chromium browser on a lite version of Raspberry Pi OS, you will need a minimum set of GUI (Graphical User Interface) components to support it.

```sudo apt-get install --no-install-recommends xserver-xorg x11-xserver-utils xinit openbox```

### 8. Install Chromium Web browser

```sudo apt-get install --no-install-recommends chromium-browser```

### 9. Edit Openbox config

* Open up autostart in an editor:

```sudo nano /etc/xdg/openbox/autostart```

* First, add commands to turn off power management, screen blanking and screen saving. We don’t want those features in a kiosk.

 ```bash
xset -dpms # turn off display power management system
xset s noblank # turn off screen blanking
xset s off # turn off screen saver
```
* Next if Chromium crashed it may pop up error messages next time it starts. This is another feature that we don’t want in a kiosk.

```shell
# Remove exit errors from the config files that could trigger a warning
sed -i 's/"exited_cleanly":false/"exited_cleanly":true/' ~/.config/chromium/'Local State'
sed -i 's/"exited_cleanly":false/"exited_cleanly":true/; s/"exit_type":"[^"]\+"/"exit_type":"Normal"/' ~/.config/chromium/Default/Preferences
```

* Finally, update autostart to run the Chromium browser in kiosk mode. Pass in an environment variable ($KIOSK_URL) that contains the URL of the Web app to launch.

```shell
# Run Chromium in kiosk mode
chromium-browser  --noerrdialogs --disable-infobars --kiosk $KIOSK_URL
```

### 10. Setup the environment

* Edit the Openbox environment file:

```sudo nano /etc/xdg/openbox/environment```

* Add the KIOSK_URL to the file:

```export KIOSK_URL=https://google.com```

### 11. Start the X server on boot

* See if ~/.bash_profile already exists:

```ls -la ~/.bash_profile```

* Add this line to start the X server on boot. Because I am using a touch screen I’m passing in the flag to remove the cursor and save the file:

```[[ -z $DISPLAY && $XDG_VTNR -eq 1 ]] && startx -- -nocursor```

* Reboot your pi

```sudo reboot```

#### Troubleshooting

##### Xorg

* if your screen is not found, past this on `/usr/share/X11/xorg.conf.d/40-libinput.conf`:

 ```bash
Section "Device"
# WaveShare SpotPear 3.5", framebuffer 1
Identifier "uga"
driver "fbdev"
Option "fbdev" "/dev/fb1"
Option "ShadowFB" "off"
EndSection

Section "Monitor"
# Primary monitor. WaveShare SpotPear 480x320
Identifier "WSSP"
EndSection

Section "Screen"
Identifier "primary"
Device "uga"
Monitor "WSSP"
EndSection

Section "ServerLayout"
Identifier "default"
Screen 0 "primary" 0 0
EndSection
```

## Screen Waveshare 3.5inch RPi LCD (A)

### 1. Install the touch driver

* Then open the terminal of Raspberry Pi to install the touch driver.

```shell
git clone https://github.com/waveshare/LCD-show.git
cd LCD-show/
```

* and: 

```shell
chmod +x LCD35-show
./LCD35-show lite
```

## Docker

### 1. Download the Convenience Script and Install Docker on Raspberry Pi

* Move on to downloading the installation script with:

```curl -fsSL https://get.docker.com -o get-docker.sh```

* Execute the script using the command:

```sudo sh get-docker.sh```

### 2. Add a Non-Root User to the Docker Group

* The syntax for adding users to the Docker group is:

```sudo usermod -aG docker [user_name]```

* To add the Pi user (the default user in Raspbian), use the command:

```sudo usermod -aG docker Pi```

### 3. Check Docker Version and Info

* Check the version of Docker on your Raspberry Pi by typing:

```docker version```

### 4. Run Hello World Container

* The best way to test whether Docker has been set up correctly is to run the Hello World container.
To do so, type in the following command:

```docker run hello-world```

### 5. Download and install node.js 14

* Install node.js 14 by first installing the required repository:

```curl -sL https://deb.nodesource.com/setup_14.x | sudo bash -```

* The script above will create apt sources list file for the NodeSource Node.js 14.x repo:

 ```shell
cat /etc/apt/sources.list.d/nodesource.list
deb https://deb.nodesource.com/node_14.x focal main
deb-src https://deb.nodesource.com/node_14.x focal main
```

* Once the repository is added, you can begin the installation of Node.js 14 on Ubuntu & Debian Linux:

```sudo apt -y install nodejs```

* test your version:

```node -v```

### 6. Download and run your app

* run ``cd Documents`` and past this following:

```git clone https://github.com/JulienChapron/covid19-leaflet-docker.git```

* run ``cd covid19-leaflet-docker``, install dependencies and build your app:

 ```shell
npm install
sudo docker build -t covid19-leaflet-docker .
```

* Create a script file

```touch /home/pi/docker.sh```

* Past this following:

 ```shell
#!/bin/bash

#run your app
docker run --rm -d  -p 8080:8080/tcp covid19-leaflet-docker:latest
```

* open ``/etc/rc.local`` and pas this following:

```bash
...
sudo bash /home/pi/docker.sh &
exit 0
```
