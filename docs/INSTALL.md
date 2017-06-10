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

## Install the media player mpv 

The mpv media player not only plays almost everything (local files, web streams, playlists, folders).

Install *mpv*

~~~~
sudo apt-get install mpv
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
$ sudo reboot
~~~~

# Configure the jukebox

Continue with the configuration in the file [`CONFIGURE.md`](CONFIGURE.md).
