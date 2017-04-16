# Raspberry Pi Setup

Setup Raspbian OS and make it easy to connect headlessly via SSH on your local network.

## Setup Raspbian OS

Format SD Card: https://www.sdcard.org/downloads/formatter_4/eula_mac/

Download "Noobs Lite" https://www.raspberrypi.org/downloads/noobs/ and copy files to SD card.

Fully hook up Pi to HDMI display and then plug it in to start installing OS.

Once machine finishes booting, pick keyboard type.

Connect to WIFI.

Pick Raspbian Lite (or Raspbian Pixel if you want a Desktop)

Login and configure SSH access.  Initial login is `pi` with password `raspberry`. Run `sudo raspi-config`. Choose `Interfacing Options` and turn on `SSH`.  While there, change `pi` password and change hostname or leave it as the default `raspberrypi`.

Reboot.

Now try to login remotely:

    ssh pi@raspberrypi.local

## Install Go

    wget https://storage.googleapis.com/golang/go1.8.1.linux-armv6l.tar.gz
    sudo tar -C /usr/local -xzf go1.8.1.linux-armv6l.tar.gz
    rm go1.8.1.linux-armv6l.tar.gz

Add this to the bottom of `~/.profile` via `vi`.

    # set PATH so it includes go
    if [ -d "/usr/local/go/bin" ] ; then
      PATH="/usr/local/go/bin:$PATH"
    fi

Log out of terminal and log back in.

References:

* https://gist.github.com/konradko/a9468beb70f0fa47766f5ebf966f24e8

## Install Docker

    curl -sSL get.docker.com | sh

    # run docker commands with sudo
    sudo docker images
