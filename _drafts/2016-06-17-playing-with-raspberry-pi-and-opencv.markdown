---
layout: post
comments: true
title:  "Playing with Raspberry PI and OpenCV"
excerpt: "Just got myself a Raspberry Pi 3 to have a play with and struggled to get it to work as per the existing (enormous) collection of blog posts. So, here I've jotted down all the steps that I used to make it all work..."
date:   2016-06-17 14:30:00
mathjax: false
---

# The kit
When you unbox your new toy (Raspberry Pi 3 in this case) you should find you have the following bits of kit, including a few that should already be around the house:
+ Raspberry Pi 3 board
+ Raspberry Pi Camera module (you did order that too didn't you?)
+ Micro SD Card (with card reader for plugging into your laptop/home computer)
+ HDMI cable
+ Old-fashioned USB Keyboard
+ A TV (with HDMI input)

When I bought my Pi it came with [Noobs](https://www.raspberrypi.org/blog/introducing-noobs/) pre-installed on the SD card - or so the documentation said! However once I connected the HDMI cable to the TV and plugged the USB keyboard into the Pi, then finally "powered up" - **I just got a coloured rainbow screen. My Pi was just a brick!**.

A quick google search will show that this *unfortunately* is quite common (wtf) so I then manually downloaded [NOOBS](https://www.raspberrypi.org/downloads/) from the Raspberry PI downloads site. Followed the instructions to wipe my SD card and put the new NOOBS version on. Powered up! This time voila! I see the NOOBS setup screen.

Now... before we go any further.

NOOBS initially shows a screen that allows you to select an operating system to install. I chose Raspbian which is a variant of Debian that most people *playing* with a Pi seem to be using.
But beware if you only have an 8Gb memory card that if you install Raspbian from NOOBS it seems to take up more space than if you actually download a Raspbian image on its own and install it onto the SD card yourself.

This caused me issues as later on we want to compile OpenCV on the Pi and it needs a good 4GB of space to do this. With the NOOBS installed Raspbian I only had about 3.8Gb free (and thats after I de-installed the Wolfram app which is nearly 1Gb on its own).

Biting the bullet:

Instead of mucking around I just went out and bought a 32GB card. They are pretty cheap now at about AUD$24 for a fast one.

So to sum up.... download the [Raspbian](https://www.raspberrypi.org/downloads/) image directly and following the [instructions](https://www.raspberrypi.org/documentation/installation/installing-images/README.md) to get it on your SD card. Then boot up the Pi.

*** WARNING ***  The Raspberry PI download site is woefully slow. I was using a 90Mbps Internet connection yet it still took a full 24 hours to download Raspbian!

# Basic config
For me the Pi initially boots into the Raspbian GUI. I don't want this so need to set it to boot to a terminal instead.

First issue I had was... How do I select the menus in the GUI (I didn't have an old-fashioned USB mouse laying around). Luckily by just clicking every key on the keyboard I found that the Ctrl key opens up the main menu and you can then use the arrow keys to navigate around.

You can select the Terminal application and from there run the Pi configuration utility with:
```
sudo raspi-config
```

So with `sudo raspi-config` open we need to perform the following tasks:
+ Extend the size of the memory card (by default Raspbian can't see the whole card so extend it to max.)
+ Set Terminal on boot (I don't want the GUI to launch each time we boot)
+ Enable the Pi Camera module.
+ Set logon on boot (I set it to ask for username and password on boot - this is not necessary but I just liked it.)
+ Enable SSH.

<hr>
By default the Pi's username is `pi` and password is `raspberry`.
<hr>

# SSH and a Static IP
We don't want to always be chained to a TV with the HDMI cable and have a keyboard plugged into our Pi so typically we will SSH into it. To make this more pleasant though, lets setup a static IP address for the Pi, otherwise we will have to stuff around with figuring out its IP address every time we turn it on.
Now there is *the old way* and there is the *new way* to do this...

The old way involved editing the /etc/network/interfaces file. Most blog posts that I've seen all tell you to do it this way (such as [this](www.modmypi.com/blog/tutorial-how-to-give-your-raspberry-pi-a-static-ip-address) one); but according to the raspberry pi forums there is *[the new way](http://raspberrypi.stackexchange.com/questions/37920/how-do-i-set-up-networking-wifi-static-ip)* which I've highlighted below...

## First we need to enable WiFi
You can run
```
sudo iwlist wlan0 scan
```
...to see a list of available WiFi networks.


Now edit the wpa-supplicant file with `sudo nano /etc/wpa_supplicant/wpa_suplicant.conf` and add the following at the bottom:
```
network={
	ssid="Your Wifi ESSID which can be seen in the iwlist command"
	psk="Your wifi password"
}
```
Now save by entering `Ctrl-O` then `Ctrl-X`.

`sudo reboot` and your Pi should now be connected to your wifi network.

## Now enable a static IP
The *old way* involved changing the /etc/network/interfaces file.
The *new way* involves changing the /etc/dhcpd.conf field instead. We need to add the following values:
```
static ip_address=192.168.1.99		# Whatever IP you want
static routers=192.168.0.1			# Gateway address
static domain_name=192.168.0.1		# Destination address
```
You can use `ifconfig` and `netstat -nr` to obtain some of these values.

You should now be able to log into your pi via the static IP.

Lets give it a go:
```
ssh pi@192.168.1.99
```

Bingo!

# Useful commands
A few useful commands before we go on:
```
sudo raspi-config		# Configuration utility program
sudo reboot				# Reboot the Pi
sudo poweroff			# Do everything necessary (like log out) so that its safe to pull out the power cable
df -h					# View file system details
kill -9 <process id>	# Kill a running process where <process id> can be determined with 'ps'
```

If you are using a terminal such as Cathode or Prompt on the iPad you will want to install **[screen](https://www.gnu.org/software/screen/manual/screen.html)** on the Pi. Without it your ssh session will terminate often (whenever you iPad sleeps or the app is in background for too long). This is a real pain as any running command (like an install) will terminate.

Installing screen is as simple as:
```
sudo apt-get install screen
```

To use it simply run `screen bash` which will open another terminal instance. You can now run processes that won't terminate if your ssh session does.
To detach from the termination screen terminal session use `Ctrl+A and the D`.

To list all screen instances use `screen -list`.

To reconnect to a screen terminal use `screen -r` (if you only have one) or `screen -r <terminal instance>`.

To end a terminal instance use `Ctrol+D` from within the instance.

# Home surveillance system
## Motion
To build your own home surveillance system with the Pi you can install the great `motion` program which runs just fine on the Pi.

[Motion](http://www.lavrsen.dk/foswiki/bin/view/Motion/WebHome) is an application for linux systems that can use a USB camera feed to detect motion and trigger specific actions - such as saving a photo or video clip.
Most of the details below came from this blog post: https://learn.adafruit.com/cloud-cam-connected-raspberry-pi-security-camera/dropbox-sync#motion-setup.

### Kernel module for Raspberry Pi camera
Motion runs great on the Pi, but unlike what all the blogs posts out there recommend; you need to activate a kernel module (VL42) in Raspbian so that the Pi camera is treated like a USB camera. Then it will work flawlessly with the Pi.

Edit the `etc/modules` file to enable the module as follows:
```
sudo nano /etc/modules
```
This file controls what kernel modules are loaded on boot. Scroll to the bottom of the file and add the following:
```
bcm2835_v4l2
```
Save the file and exit.

Now `sudo reboot`

After the Pi starts again connect via SSH and run the following command to check the module loaded successfully:
```
ls -l /dev/video*
```
If you don't see the `/dev/video0` source listed you can run the `lsmod` command to list all the active kernel modules and check if the `bcm2835_v4l2` module is loaded. If not you can attempt to manually load the module by running `sudo modprobe bcm2835_v4l2`. If all else fails you can check the kernel log by running `dmesg` to see if there are any relevant error messages. Other than that you'll have to try Google.

### Install
To install motion on the PI simply run:
```
sudo apt-get update
sudo apt-get install -y motion
```

Fix issue with ownership of location where motion stores images:
```
sudo mkdir /var/lib/motion
sudo chown motion:motion /var/lib/motion
```

Motion is not installed. We just need to *configure* it to do what we want now...

### Configure
To configure motion you simply edit the motion.conf file.
```
sudo nano /etc/motion/motion.conf
```

Most options you'll want to leave as default, but a few that I changed we as follows:
+ width 1280
+ height 720
+ threshold 3000
+ minimum_motion_frames 2
+ ffmpeg_output_movies off (I turned this off as I just wanted to capture jpeg stills)
+ stream_localhost off (**Make sure to turn this OFF else you won't be able to see a video stream in your laptops browser**)
+ on_picture_save => Set this to a script you wish to run on saving a picture - it is disabled by default.

*Check the [motion](http://www.lavrsen.dk/foswiki/bin/view/Motion/WebHome) website for a list of all available options*

**Note that by default motion stores pictures/videos in here: /var/lib/motion.**

I added a script to the *on_picture_save* config to save all the jpegs to Dropbox. On a *motion* event you can have it run any script... to literally do anything. Send email, SMS, etc.

### Setup motion as a service
Motions can be setup as a service so that its always on and starts on boot. To enable this we need to edit the `/etc/default/motion`:
```
sudo nano /etc/default/motion
```
In here you can set yes or no to enable motion as a daemon. If enabled you can use the following to control the motion service:
```
sudo service motion stop
sudo service motion restart
sudo service motion start
```
Note: With motion configured as a daemon it will auto-start on Pi boot.

With the config we did above you can see a live stream from your camera by pointing your browser to the Raspberry Pi's IP address and port 8081: `192.168.1.99:8081`

## Roll your own motion detection
As motion is a *black box* and we'd really like to play with our own motion detection code and some machine learning techniques to see if we can do *better* than motion, next we'll install the OpenCV library and write some of our own code...

# Install OpenCV 3 so we can play with the Pi camera
This will require compiling OpenCV source on the Pi and will require a minimum of around 4GB of space.

When building from source **do not** use the `make -j4` option (which uses all 4 cores) as it failed every time for me. Instead just use `make` which may be a bit slower - **but it works**.
The build time was approx. 3 hours for me.

## X Server for viewing application windows
If you wish to view any graphical applications that are running on the Pi, but you typically run the Pi in headless mode (no screen connected) then you can make use of X Windows.

SSH will allow you to connect with X-forwarding enabled. If your laptop us running linux it probably has an X server running by default. If however, you are on Windows or Mac OSX (like I am) then you need to install one.

For Windows I've read that there are a few options you can choose from but as I don't use Windoze - you're on your own.
For OSX there is [XQuartz](https://www.xquartz.org/). Previously OS came with an X11 server but Apple decided to drop it and spun it off into an open source project.
Just download the DMG from https://www.xquartz.org/ and install it. If you then connect to the Pi via SSH with X-forwarding enabled your apps windows will magically start appearing on your laptop.

[*Note that X11 work with the keyboard and mouse as well, not just the screen. Also - slightly confusing - the X Server runs on the device that has the physical screen and keyboard.*]

Connect via SSH with X-forwarding:
```
ssh -X pi@192.168.1.99
```
 TODO - Add some pics here!!!

## Accessing the pi camera
OpenCV works with USB webcams. This includes when running it on the Raspberry Pi. However, the Pi Camera is *not* a USB webcam.

We looked at one way to cater for this when using motion, and that was to install a kernel extension that enables the Pi camera to be treated as a USB camera. In general however if we are writing our own code with OpenCV we don't need the kernel extension as there is a great python module available called picamera. picamera is nice because it gives you complete control of the camera and its settings.

Even better... There is a set of [image processing utilities](https://github.com/jrosebr1/imutils) available that provides a nice wrapper class which can be used in your OpenCV code without worrying which camera you are using (whether it be the Pi camera or a USB camera - of for that matter the camera on your laptop).
If we use this wrapper our code can work regardless of the camera type.

See this pyimagesearch.com blog post for an example on how to use this camera wrapper class: [imutils wraper class for camera](http://www.pyimagesearch.com/2016/01/04/unifying-picamera-and-cv2-videocapture-into-a-single-class-with-opencv/)

# Next...
Now that we have the camera up and running on our pi, I'm going to switch  to working on my laptop. When writing code its just waaaaay easier to play on the laptop than directly on the pi. We can get our image processing code working and then when happy, move it to the Pi.
