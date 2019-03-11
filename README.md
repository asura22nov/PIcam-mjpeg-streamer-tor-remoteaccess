# PIcam-mjpeg-streamer-tor-remoteaccess
#RaspberryPI – 3B – Board
#Pi – NoIR Camera V2 – Camera 
#Remote Monitoring via TOR network through Mjpeg-Streamer 
#(Without Static IP requirement & Dyn DNS)

###################
#CAUTION:
#Only for people have have prior knowledge in 
#raspberry pi, 
#linux commands, 
#windows operating system
#Apache working in Raspberry
#PIcam Camera Module
#Tor Hidden Service in windows and Raspberry
#Working of Xming in windows
#Working of Putty
#
#
###################
#Requirements

#Hardware:
#1.Raspberry PI 3B (8 / 16 GB SD card) + Power Adaptor
#2.Pi NoIR Camera V2
#3.WIFI or 3G/4G Dongle

#Software:
#For Raspberry PI 3B
#1.Raspbian - Linux raspberrypi 4.14.79-v7+
#2.Tor version 0.3.5.7.
#3.Apache
#       a.Server version: Apache/2.4.25 (Raspbian)
#       b.Server built:   2018-11-03T18:46:19
#4.mjpg-streamer
#5.feh  2.18-2 (imlib2 based image viewer)
#6.Linux camera module bcm2835-v4l2s
	
#For Windows Testing
#1.Putty
#2.Xming Server for (Windows Desktop)

#Installations:
sudo apt-get update -y
sudo apt-get install apache2 –y
sudo apt-get install cmake libjpeg8-dev

#Raspberry PI Systmectl commands:
sudo systemctl start apache2.service
sudo systemctl status apache2.service

#Procedure:

#1)Checking the Pi Camera Working
#command to load linux camera module during boot-up

sudo nano /etc/modules

#enter 'bcm2835-v4l2'
#save and reboot

#To check wheather the module has loaded, type:
lsmod

#if not loaded, then:

sudo modprobe bcm2835-v4l2s

#Test & Checking:

#From Windows CMD, connect to your raspberry
#make sure you have started your Xming program

putty.exe -ssh –X 192.168.0.3-p 22

#after logging into your raspberry

raspistill -t 2000 -o image-640-480.jpg -w 640 -h 480

#issue the below command to view the pick from your windows laptop

feh image-640-480.jpg

#to remove the pic issue the below cmd in raspberry:

rm image-640-480.jpg

#2)download MJPG STREAMER and Check for the Mjpeg-Streamer Working

wget https://github.com/jacksonliam/mjpg-streamer/archive/master.zip

unzip master.zip

cd mjpg-streamer-master/

cd mjpg-streamer-experimental/

make clean

cmake -DPLUGIN_INPUT_OPENCV=OFF

make

sudo make install

#issue the below cmd to check the version of the mjpg-streamer installed in raspberry

which mjpg_streamer

#File Paths:

ls -l /usr/local/bin/

ls -l /usr/local/lib/mjpg-streamer directory

ls -l /usr/local/share/mjpg-streamer/www

#Command line:

/usr/local/bin/mjpg_streamer -i "/usr/local/lib/mjpg-streamer/input_uvc.so -n -f 10 -r 1280x720" -o "/usr/local/lib/mjpg-streamer/output_http.so -p 8085 -w /usr/local/share/mjpg-streamer/www"

#Automation:

nano /etc/init.d/mjpeg-streamer.sh

#!/bin/bash
/usr/local/bin/mjpg_streamer -i "/usr/local/lib/mjpg-streamer/input_uvc.so -n -f 10 -r 1280x720" -o "/usr/local/lib/mjpg-streamer/output_http.so -p 8085 -w /usr/local/share/mjpg-streamer/www"

sudo update-rc.d mjpeg-streamer.sh defaults

#verify whether you have created a auto-script in RC folder:

cd /etc/rc2.d/

ln –s /etc/init.d/mjpeg-streamer.sh  /etc/rc2.d/S01mjpeg-streamer.sh

ls –lrt /etc/rc2.d/

S01mjpeg-streamer.sh -> /etc/init.d/mjpeg-streamer.sh

#From Windows Browser:

http://192.168.0.3:8085/stream_simple.html


http://192.168.0.3:8085/index.html


http://192.168.0.3:8085/control.htm


#3)Check for the Apache Working

#3.1

/etc/apache2/sites-available

sudo cp 000-default.conf mjpeg-streamer.conf

#3.2

sudo nano /etc/apache2/sites-available/mjpeg-streamer.conf

#make the below modification as shown below

<VirtualHost *:8085>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /usr/local/share/mjpg-streamer/www/

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet

#3.3

sudo a2ensite mjpeg-streamer.conf && sudo systemctl restart apache2.service

#to disable

sudo a2dissite mjpeg-streamer.conf

#Enabling site mjpeg-streamer.conf,
#To activate the new configuration, you need to run:

systemctl reload apache2

#5)Restart the apache web server

sudo systemctl restart apache2.service

#4)Check for the TOR Working

sudo nano /etc/tor/torrc

# add the below lines to create a new hidden service, make sure you run tor as user with sudo rights
#HiddenServiceDir /home/tor-hidden/mjpg-streamer/
#HiddenServicePort 80 127.0.0.1:8085

sudo systemctl restart tor.service

#Tor HiddenService, just an exmple (its not a workable link):
http://afanfajsfafhwefih.onion/static.html

#Reference:
#•	https://jacobsalmela.com/2014/05/31/raspberry-pi-webcam-using-mjpg-streamer-over-internet/
#
#•	https://github.com/jacksonliam/mjpg-streamer/issues/160
#
#•	https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-phpmyadmin-on-ubuntu-12-04
#
#•	https://gk2.sk/running-ssh-on-a-raspberry-pi-as-a-hidden-service-with-tor/
#
#•	https://trac.torproject.org/projects/tor/wiki/doc/TorifyHOWTO/ssh
#
#•	https://www.torproject.org/docs/tor-onion-service.html.en
