# Raspberry Pi Setup

Setup Raspbian OS and make it easy to connect to via SSH on your local network.

Tested on Raspberry Pi Zero W Rev 1.1 (and Raspberry Pi 3).

## Setup Raspbian OS

Download Raspbian Stretch Lite (https://www.raspberrypi.org/downloads/raspbian/).

Burn Raspbian `.img` file to SD card using Etcher (https://etcher.io/) for macOS.

Insert SD card into Raspberry Pi. Fully hook up Pi to HDMI display and keyboard, then plug it in to start installing OS.

Once machine finishes booting, initial login is `pi` with password `raspberry`.

Run `sudo raspi-config`.

Setup keyboard. This is important, otherwise  you may type wrong characters when setting up Wi-fi next. Choose `Localization` menu. Choose `Change Keyboard Layout` menu. If your exact keyboard is not on the list then choose one of the generic 101, 102 or 104 keyboards.  (Mine was 101 Generic -- if in US, don't pick default English UK -- choose Other and pick English US)

Connect to Wi-fi via `Network Options` > `Wi-fi`. While there, change `pi` password and change hostname or leave it as the default `raspberrypi`.

If this doesn't work, `sudo nano /etc/wpa_supplicant/wpa_supplicant.conf` and make sure settings look ok. Restart network via `wpa_cli -i wlan0 reconfigure`. (https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md)

Choose `Update` menu to test wifi and get latest files.

Then enable SSH access. Choose `Interfacing Options` and turn on `SSH`.

Optionally, change Hostname via `Network` > `Hostname` menu.

Change password via `Network` > `Change User Password`.

Expand filesystem via `Advanced` > `Expand Filesystem` menu.

Exit `raspi-config` and choose to Reboot.

## Connect to your Pi remotely via SSH

Now try to login remotely:

    ssh pi@raspberrypi.local

If you can't find your Pi on the network

    brew install nmap
    sudo nmap -sP 192.168.1.0/24 | awk '/^Nmap/{ip=$NF}/B8:27:EB/{print ip}'

where 192.168.1.* will be your local network mask.  More details: https://raspberrypi.stackexchange.com/questions/13936/find-raspberry-pi-address-on-local-network

If that all works, you can now unplug your peripherals and connect to the Raspberry PI via SSH.

## Setup passwordless login

    mkdir ~/.ssh
    chmod 700 ~/.ssh
    touch ~/.ssh/authorized_keys
    chmod 600 ~/.ssh/authorized_keys
    vi ~/.ssh/authorized_keys
    # add your id_rsa.pub key in here

# CLI OPTION

## Setup autologin with CLI

From https://www.raspberrypi.org/forums/viewtopic.php?t=127042:

Create `/etc/systemd/system/getty@tty1.service.d/autologin.conf`

And put this in it:

```
[Service]
ExecStart=
ExecStart=-/sbin/agetty --autologin pi --noclear %I 38400 linux
```

Run `sudo systemctl enable getty@tty1.service`

## Create script to run slideshow

Create `~/show.sh` to change image every 60 seconds:

```
#!/bin/bash
fbi -noverbose -m 1280x800 -u -a -T 1 -t 60 /home/pi/*.JPG
```

## Auto run slideshow

In `~/.bashrc` add:

`sudo bash /home/pi/show.sh`

Note: Only problem with this is that each time you ssh in, it restarts the slidenow.

# DESKTOP OPTION (run Chromium in Kiosk mode)

Install a GUI from Raspbian Lite (https://www.raspberrypi.org/forums/viewtopic.php?t=133691)

    #sudo apt-get install --no-install-recommends xinit raspberrypi-ui-mods lxterminal gvfs
    sudo apt-get install --no-install-recommends raspberrypi-ui-mods
    sudo apt-get install -y chromium-browser unclutter
    sudo reboot now

    # from: https://raspberrypi.stackexchange.com/questions/40631/setting-up-a-kiosk-with-chromium
    cp /etc/xdg/lxsession/LXDE-pi/autostart /home/pi/.config/lxsession/LXDE-pi/autostart
    sudo vi /home/pi/.config/lxsession/LXDE-pi/autostart

```
#@xscreensaver -no-splash  # comment this line out to disable screensaver
@xset s off
@xset -dpms
@xset s noblank
@sed -i 's/"exited_cleanly": false/"exited_cleanly": true/' ~/.config/chromium/Default/Preferences
@chromium-browser --incognito --noerrdialogs --kiosk https://www.briangershon.com/
```

    sudo reboot now
    # you should now see the above webpage in full screen

## Install Go

    wget https://storage.googleapis.com/golang/go1.9.2.linux-armv6l.tar.gz
    sudo tar -C /usr/local -xzf go1.9.2.linux-armv6l.tar.gz
    rm go1.9.2.linux-armv6l.tar.gz

Add this to the bottom of `~/.profile` via `vi`.

```
# set PATH so it includes go
if [ -d "/usr/local/go/bin" ] ; then
  PATH="/usr/local/go/bin:$PATH"
fi
```

Log out of terminal and log back in. Try `go version` to make sure it's running ok.

## Install Docker

    # NOTE that latest Docker (17.11.0) is broken on Raspberry Pi. Until fixed run this:
    sudo apt install docker-ce=17.09.0~ce-0~raspbian

    # Otherwise, run:
    sudo apt-get install docker-ce

    # try it
    sudo docker run hello-world

If you would like to use Docker as a non-root user, you should now consider adding your user to the "docker" group with something like: `sudo usermod -aG docker pi`
