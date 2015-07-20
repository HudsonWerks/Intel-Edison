SETTING UP DEBIAN ON EDISON


1. Set up and run Virtual Machine

2. Running Ubuntu 12.04

3. To get VM to recognize Edison as a USB device, poweroff Edison then reboot


Some additional, helpful details on SparkFun guide: https://learn.sparkfun.com/tutorials/loading-debian-ubilinux-on-the-edison.

HOWEVER, FROM HERE, MAINLY USING:
	a) http://www.emutexlabs.com/ubilinux/29-ubilinux/218-ubilinux-installation-instructions-for-intel-edison

	b) Also useful: https://communities.intel.com/thread/59312


4. Running flashall.sh:
	Plug and reboot the board.

5. Unplugged both USB cables and the 5V supply. Then, plugged USB cables back in, then the 5V.
	flashall.sh seemed to run more normally

TROUBLESHOOTING:
	on VM/Ubuntu term:
	# screen /dev/tty/USB0 115200

6. After running flashall.sh and letting script finish completely (following status on screen), shutdown Edison AND VM.

7. Reboot Edison and it should now appear on desktop

8. creds:
	default UID: root
	default PW: edison

9. Modify network interfaces. Make sure to un-comment the auto wlan0 line to have wlan come up at boot.: 

	# nano /etc/network/interfaces

# interfaces(5) file used by ifup(8) and ifdown(8)
auto lo
iface lo inet loopback

auto usb0
iface usb0 inet static address 192.168.2.15 netmask
255.255.255.0

auto wlan0
iface wlan0 inet dhcp
    # For WPA
    wpa-ssid <network name>
    wpa-psk <password>
#    wpa-psk passphrase
    # For WEP
    #wireless-essid Emutex
    #wireless-mode Managed
    #wireless-key s:password

	Then:
	# ifup wlan0

ADDED THESE PKGS
	1) followed suggestions from http://wiki.ros.org/wiki/edison
	# apt-get install sudo screen

	# useradd <user_name>
	# chown <user_name>:users /home/<user_name>
	# passwd <password>
	 AT PW PROMPT: <password>
	# echo "<user_name>  ALL=(ALL:ALL) ALL" >> /etc/sudoers
	# chsh <user_name> -s /bin/bash

	CONFUSED BY THIS STEP:
	Edit /etc/hosts and add "ubilinux" to the localhosts line. (If you don't do this, sudo won't work) 


	MISC:

	VNCSERVER
	# apt-get install tightvncserver

	Xsession: unable to start X session --- no "/root/.xsession" file, no "/root/.Xsession" file, no session managers, no window managers, and no 
terminal emulators found; aborting.

		TROUBLESHOOTING: NO X11 on EDISON
	# apt-get install xorg openbox
	# apt-get install xauth
	# apt-get install x11-xserver-utils
	

	ADDED THIS TO /ETC FILE (CHECK EXACT FILE NAME)
	127.0.0.1       localhost ubilinux
	::1 localhost ip6-localhost ip6-loopback
	fe00::0 ip6-localnet ff00::0 ip6-mcastprefix ff02::1 ip6-allnodes
	ff02::2 ip6-allrouters

		XRDP
		# apt-get install xrdp

		SCREEN SHARING
		In Mac terminal (not logged into Edison):
			$ open vnc://root@192.168.15.128:5901

		PROBABLY THE ONLY PKG THAT I NEEDED:
		# apt-get install x11vnc

		# x11vnc -display :1 <change display value if necessary>
		STARTUP VNC SCRIPTS AND FURTHER INFO:
			- Includes startup script for RPi - http://www.penguintutor.com/raspberrypi/tightvnc
			- Includes script - http://askubuntu.com/questions/229989/how-to-setup-x11vnc-to-access-with-graphical-login-screen
			- Includes bash script - http://www.karlrunge.com/x11vnc/
			- using TKinter and VNC on RPi - http://stackoverflow.com/questions/20286705/tkinter-through-vnc-without-physical-display
			- other RPi VNC guidance - https://pihw.wordpress.com/guides/guide-to-remote-connections/


	SIP CLIENT
	# apt-get install linphone
	(installed current repo 3.5.2, which is two versions behind source compile version)

		LINPHONE CMD INFO
		- http://www.linphone.org/technical-corner/linphone/documentation
		- https://github.com/nerdOmat/RaspyPhone/wiki/Software
		- https://github.com/nerdOmat/RaspyPhone/blob/master/voip-software.py		

	MISC
	# dpkg-reconfigure tzdata”
	# apt-get install less
	# apt-get install curl
	# apt-get install python-pip
	# apt-get install libcurl4-openssl-dev
	# apt-get install python-distutils-extra
	# pip install pycurl


	NODE (Supposed to be installed with UbiLinux image, but wasn't)
	# apt-get install curl
	# curl -sL https://deb.nodesource.com/setup | bash -
	# apt-get install nodejs

	MRAA LIB (Supposed to be installed with UbiLinux image, but wasn't)
	# apt-get install git cmake swig ntp

	# cd mraa
	# cd build
	# cmake .. -DBUILDSWIGNODE=OFF -DCMAKE_INSTALL_PREFIX:PATH=/usr
	# make
	# make install

	CYLON (for fast proto SWITCH code)
	npm install cylon cylon-intel-iot


AUDIO
USE THIS AS REFERENCE:
http://alextgalileo.altervista.org/blog/lets-make-noise-play-audio-edison/

	# apt-get install alsa-base libasound2 libasound2-dev alsa-tools alsa-utils	

	# apt-get install pulseaudio
	# apt-get install pulseaudio-utils


	TEST PLAYBACK
	# aplay /usr/share/sounds/alsa/Front_Center.wav


INCREASE SIZE OF ROOTFS
Workable, though not great since it breaks the repo caches (from this https://communities.intel.com/message/274839#274839):

	# mv /var/cache /home
	# cd /var
	# ln -sf /home/cache cache
	# df -ah

	Since the above breaks the repo caches, you have to fix this, so run these steps:
		# mkdir -p /var/cache/apt/archives/partial
		# touch /var/cache/apt/archives/lock
		# chmod 640 /var/cache/apt/archives/lock
		# apt-get update


BLUETOOTH TROUBLESHOOTING

	Re-compiled Bluez
	# ./configure
	# ./configure --disable-systemd
	# make -j 2
	# make install

	Version:
	# dpkg --status bluez | grep '^Version:'

		OR

	# bluetoothd -v


	TRY THIS, even THOUGH FOR YOCTO:
	https://communities.intel.com/thread/61548?start=15&tstart=0
	

HEADLESS EDISON
http://tobyho.com/2015/01/09/headless-browser-testing-xvfb/
http://e-method.blogspot.fr/2010/11/google-chrome-with-xvfb-headless-server.html

REVERT TO YOCTO
https://learn.sparkfun.com/tutorials/loading-debian-ubilinux-on-the-edison
