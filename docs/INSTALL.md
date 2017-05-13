# Installing the RFID Jukebox on your RPi

The installation is the first step to get your jukebox up and running. Once you have done this, proceed to the [configuration](CONFIGURE.md).

And Once you finished with the configuration, read the [manual](MANUAL.md) to add audio files and RFID cards.

This project has been tested on Raspberry Pi model 1 and 2. And there is no reason why it shouldn't work on the third generation.

## Install Raspbian on your RPi

### Burning NOOBS to the SD-card

There are a number of operating systems to chose from on the [official RPi download page](https://www.raspberrypi.org/downloads/) on [www.raspberrypi.org](http://www.raspberrypi.org). We want to work with is the official distribution *Raspbian*. The easiest way to install *Raspbian* to your RPi is using the *NOOBS* distribution, which you can find on the download page linked above.

On the *NOOBS* download page, there are two flavours of the operating system. Download the version *offline and network install*.

After you downloaded the `zip` file, all you need to do is `unzip` the archive, format a SD-card in `FAT32` and move the unzipped files to the SD-card. For more details, see the [official NOOBS installation page](https://www.raspberrypi.org/learning/software-guide/quickstart/).

### Installing Raspbian OS after booting your RPi

Before you boot your RPi for the first time, make sure that you have all the external devices plugged in. What we need at this stage:

1. An external monitor connected over HDMI
2. A WiFi card over USB (unless you are using a third generation RPi with an inbuilt WiFi card).
3. A keyboard and mouse over USB.

After the boot process has finished, you can select the operating system we want to work with: *Raspbian*. Select the checkbox and then hit **Install** above the selection.

## Configure your RPi

Now you have installed and operating system and even a windows manager (called Pixel on Raspbian). Start up your RPi and it will bring you straight to the home screen. Notice that you are not required to log in.

### Configure your keyboard

In the dropdown menu at the top of the home screen, select:

'Berry' > Preferences > Mouse and Keyboard Settings

Now select the tab *Keyboard* and then click *Keyboard Layout...* at the bottom right of the window. From the list, select your language layout.

### Configure the WiFi

At the top right of the home screen, in the top bar, you find an icon for *Wireless & Wired Network Settings*. Clicking on this icon will bring up a list of available WiFi networks. Select the one you want to connect with and set the password.

**Disable WiFi power management**

Make sure the WiFi power management is disabled to avoid dropouts. [Follow these instructions](https://gist.github.com/mkb/40bf48bc401ffa0cc4d3#file-gistfile1-md).

### Access over SSH

SSH will allow you to log into the RPi from any machine in the network. This is useful because once the jukebox is up and running, it won't have a keyboard, mouse or monitor attached to it. Via SSH you can still configure the system and make changes - if you must.

Open a terminal to star the RPi configuration tool.

~~~~
$ sudo raspi-config
~~~~
Select `Interface Options` and then `SSH Enable/Disable remote command line...` to enable the remote access.

Find out more about how to [connect over SSH from Windows, Mac, Linux or Android on the official RPi page](https://www.raspberrypi.org/documentation/remote-access/ssh/).

### Autologin after boot

When you start the jukebox, it needs to fire up without stalling at the login screen. This can also be configured using the RPi config tool.

Open a terminal to star the RPi configuration tool.

~~~~
$ sudo raspi-config
~~~~

Select `Boot options` and then `Desktop / CLI`. The option you want to pick is `Console Autologin - Text console, automatically logged in as 'pi' user`.

### Set a static IP address for your RPi

To be able to log into your RPi over SSH from any machine in the network, you need to give your machine a static IP address.

Check if the DHCP client daemon (DHCPCD) is active.
~~~~
sudo service dhcpcd status
~~~~
If you don't get any status, you should start the `dhcpcd` daemon:
~~~~
sudo service dhcpcd start
sudo systemctl enable dhcpcd
~~~~
Check the IP address the RPi is running on at the moment:
~~~~
$ ifconfig

wlan0     Link encap:Ethernet  HWaddr 74:da:38:28:72:72  
          inet addr:192.168.178.82  Bcast:192.168.178.255  Mask:255.255.255.0
          ...
~~~~
You can see that the IP address is 192.168.178.82. We want to assign a static address 192.168.178.199.

**Note:** assigning a static address can create conflict with other devices on the same network which might get the same address assigned. Therefore, if you can, check your router configuration and see if you can assign a range of IP addresses for static use.

Change the IPv4 configuration inside the file `/etc/dhcpcd.conf`.
~~~~
sudo nano /etc/dhcpcd.conf
~~~~
In my case, I added the following lines to assign the static IP. You need to adjust this to your network needs:

~~~~
interface wlan0
static ip_address=192.168.178.199/24
static routers=192.168.178.1
static domain_name_servers=192.168.178.1
~~~~
Save the changes with `Ctrl & O` then `Enter` then `Ctrl & X`.

## Install samba to share folders over your home network

To make the jukebox easy to administer, it is important that you can add new songs and register new RFID cards over your home network. This can be done from any machine. The way to integrate your RPi into your home network is using *Samba*, the standard [Windows interoperability suite for Linux and Unix](https://www.samba.org/).

Open a terminal and install the required packages with this line:

~~~~
$ sudo apt-get install samba samba-common-bin 
~~~~

First, let's edit the *Samba* configuration file and define the workgroup the RPi should be part of.

~~~~
$ sudo nano /etc/samba/smb.conf
~~~~

Edit the entries for workgroup and wins support:

~~~~
workgroup = WORKGROUP
wins support = yes
~~~~

If you are already running a windows home network, add the name of the network where I have added `WORKGROUP`. 

Now add the specific folder that we want to be exposed to the home network in the `smb.conf` file. 

~~~~
[pi_jukebox]
   comment= Pi Jukebox
   path=/home/pi/RPi-Jukebox-RFID/shared
   browseable=Yes
   writeable=Yes
   only guest=no
   create mask=0777
   directory mask=0777
   public=no
~~~~

**Note:** the `path` given in this example works (only) if you are installing the jukebox code in the directory `/home/pi/`.

Finally, add the user `pi` to *Samba*. For simplicity and against better knowledge regarding security, I suggest to stick to the default user and password:

~~~~
user     : pi
password : raspberry
~~~~

Type the following to add the new user:

~~~~
$ sudo smbpasswd -a pi
~~~~

## Adding python libraries

### Installing evdev

In order to read the IDs from the RFID cards, we need to dig deep into the operating system. We need to have an ear at the source of the RFID reader, so to speak. And in order to listen to these events using the programming language *python*, we need to [install the package *evdev*. [Try the official installation procedure first](http://python-evdev.readthedocs.io/en/latest/install.html). If you run into problem, like I did, this might work:

~~~~
$ sudo apt-get install python-dev python-pip gcc
~~~~

Find out the linux kernel release you are running:

~~~~
$ uname -r
4.4.34+
~~~~

This means you are running release `4.4.34+`. Knowing this information, install the linux headers for your linux kernel by using the first to numbers of the release, in this case:

~~~~
$ sudo apt-get install linux-headers-4.4
~~~~

Now the system is ready to load the important package for the python code we use: *evdev*. 

~~~~
$ sudo pip install evdev
~~~~

## Running the web app

There is a second way to control the RFID jukebox: through the browser. You can open a browser on your phone or computer and type in the static IP address that we assigned to the RPi earlier. As long as your phone or PC are connected to the same WiFi network that the RPi is connected to, you will see the web app in your browser.

### Installing lighttpd and PHP

~~~~
$ sudo apt-get install lighttpd php5-common php5-cgi php5
~~~~

### Configuring lighttpd

Open the configuration file:

~~~~
$ sudo nano /etc/lighttpd/lighttpd.conf
~~~~

Change the document root, meaning the folder where the webserver will look for things to display or do when somebody types in the static IP address. To point it to the Jukebox web app, change the line in the configuration to:

~~~~
server.document-root = "/home/pi/RPi-Jukebox-RFID/htdocs"
~~~~

The webserver is usually not very powerful when it comes to access to the system it is running on. From a security point of view, this is a very good concept: you don't want a website to potentially change parts of the operating system which should be locked away from any public access.

We do need to give the webserver more access in order to run a web app that can start and stop processes on the RPi. To make this happen, we need to add the webserver to the list of users/groups allowed to run commands as superuser. To do so, open the list of sudo users in the nano editor:

~~~~
$ sudo nano /etc/sudoers
~~~~

And at the bottom of the file, add the following line:

~~~~
www-data ALL=(ALL) NOPASSWD: ALL
~~~~

The final step to make the RPi web app ready is to tell the webserver how to execute PHP. To enable the lighttpd server to execute php scripts, the fastcgi-php module must be enabled.

~~~~
$ sudo lighty-enable-mod fastcgi-php
~~~~

Now we can reload the webserver with the command:

~~~~
$ sudo service lighttpd force-reload
~~~~

Next on the list is the media player which will play the audio files and playlists: VLC. In the coming section you will also learn more about why we gave the webserver more power over the system by adding it to the list of `sudo` users.

## Install the media player VLC 

The VLC media player not only plays almost everything (local files, web streams, playlists, folders), it also comes with a command line interface `CLVC` which we will be using to play media on the jukebox.

Install *VLC*

~~~~
sudo apt-get install vlc
~~~~

Ok, the next step is a severe hack. Quite a radical tweak: we will change the source code of the VLC binary file. We need to do this so that we can control the jukebox also over the web app. VLC was designed not to be run with the power of a superuser. In order to trigger VLC from the webserver, this is exactly what we are doing.

Changing the binary code is only a one liner, replacing `geteuid` with `getppid`. If you are interested in the details what this does, you can [read more about the VLC hack here](https://www.blackmoreops.com/2015/11/02/fixing-vlc-is-not-supposed-to-be-run-as-root-sorry-error/).

~~~~
$ sudo sed -i 's/geteuid/getppid/' /usr/bin/vlc
~~~~

**Note:** changing the binary of VLC to allow the program to be run by the webserver as a superuser is another little step in a long string of potential security problems. In short: the jukebox is a perfectly fine project to run for your personal pleasure. It's not fit to run on a public server.

## Install VLC-CTRL

A command line utility to control a running vlc player instance.
Lets you control a running instance of vlc player, start a new instance, get track information, etc.

~~~~
$ sudo apt-get install python-dbus
$ sudo pip install vlc-ctrl
~~~~

## Install mpg123

While we are using *VLC* for all the media to be played on the jukebox, we are using the command line player *mpg123* for the boot sound. More about the boot sound in the file [`CONFIGURE.md`](CONFIGURE.md). To install this tiny but reliable player, type:

```
$ sudo apt-get install mpg123
```

## Install git

[*git* is a version control system](https://git-scm.com/) which makes it easy to pull software from GitHub - which is where the jukebox software is located.

~~~~
$ sudo apt-get update
$ sudo apt-get install git
~~~~

## Install the jukebox code

~~~~
$ cd /home/pi/
$ git clone https://github.com/hankman13/RPi-Jukebox-RFID.git
~~~~

## Reboot your Raspberry Pi

Ok, after all of this, it's about time to reboot your jukebox. Make sure you have the static IP address at hand to login over SSH after the reboot.

~~~~
sudo reboot
~~~~

# Configure the jukebox

Continue with the configuration in the file [`CONFIGURE.md`](CONFIGURE.md).
