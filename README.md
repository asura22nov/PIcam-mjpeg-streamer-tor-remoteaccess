# PIcam-mjpeg-streamer-tor-remoteaccess

RaspberryPI – 3B – Pi – NoIR Camera V2 – Remote Monitoring – Via TOR – Mjpeg-Streamer (Without Static IP requirement & Dyn DNS)

Requirements

Hardware:
1.	Raspberry PI 3B (8 / 16 GB SD card) + Power Adaptor
2.	Pi NoIR Camera V2
3.	WIFI or 3G/4G Dongle

Software:
For Raspberry PI
1.	Raspbian - Linux raspberrypi 4.14.79-v7+
2.	Tor version 0.3.5.7.
3.	Apache
a.	Server version: Apache/2.4.25 (Raspbian)
b.	Server built:   2018-11-03T18:46:19
4.	mjpg-streamer
5.	feh  2.18-2 (imlib2 based image viewer)
6.	Linux camera module bcm2835-v4l2s
7.	
For Windows
1.	Putty
2.	Xming Server for (Windows Desktop)

Installations:
sudo apt-get update -y
sudo apt-get install apache2 –y
sudo apt-get install cmake libjpeg8-dev

sudo systemctl start apache2.service
sudo systemctl status apache2.service

Procedure:

1)	Checking the Pi Camera Working
sudo nano /etc/modules

bcm2835-v4l2

save and reboot

lsmod

or  sudo modprobe bcm2835-v4l2s

Test & Checking:

From Windows CMD

putty.exe -ssh –X 192.168.0.3-p 22

raspistill -t 2000 -o image-640-480.jpg -w 640 -h 480

feh image-640-480.jpg

rm image-640-480.jpg


2)	Check for the Mjpeg-Streamer Working

wget https://github.com/jacksonliam/mjpg-streamer/archive/master.zip

unzip master.zip

cd mjpg-streamer-master/

cd mjpg-streamer-experimental/

make clean

cmake -DPLUGIN_INPUT_OPENCV=OFF

make

sudo make install

which mjpg_streamer

File Paths:

ls -l /usr/local/bin/

ls -l /usr/local/lib/mjpg-streamer directory

ls -l /usr/local/share/mjpg-streamer/www

Command line:

/usr/local/bin/mjpg_streamer -i "/usr/local/lib/mjpg-streamer/input_uvc.so -n -f 10 -r 1280x720" -o "/usr/local/lib/mjpg-streamer/output_http.so -p 8085 -w /usr/local/share/mjpg-streamer/www"

Automation:

nano /etc/init.d/mjpeg-streamer.sh
#!/bin/bash
/usr/local/bin/mjpg_streamer -i "/usr/local/lib/mjpg-streamer/input_uvc.so -n -f 10 -r 1280x720" -o "/usr/local/lib/mjpg-streamer/output_http.so -p 8085 -w /usr/local/share/mjpg-streamer/www"

sudo update-rc.d mjpeg-streamer.sh defaults


cd /etc/rc2.d/

ln –s /etc/init.d/mjpeg-streamer.sh  /etc/rc2.d/S01mjpeg-streamer.sh

ls –lrt /etc/rc2.d/

S01mjpeg-streamer.sh -> /etc/init.d/mjpeg-streamer.sh

From Windows Browser:

http://192.168.0.3:8085/stream_simple.html


http://192.168.0.3:8085/index.html


http://192.168.0.3:8085/control.htm


3)	Check for the Apache Working

1
/etc/apache2/sites-available
sudo cp 000-default.conf mjpeg-streamer.conf

2
sudo nano /etc/apache2/sites-available/mjpeg-streamer.conf

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

3
sudo a2ensite mjpeg-streamer.conf && sudo systemctl restart apache2.service
sudo a2dissite mjpeg-streamer.conf (to disable)


Enabling site mjpeg-streamer.conf,
To activate the new configuration, you need to run:
  systemctl reload apache2

5
sudo systemctl restart apache2.service

4)	Check for the TOR Working


sudo nano /etc/tor/torrc

HiddenServiceDir /home/tor-hidden/mjpg-streamer/
HiddenServicePort 80 127.0.0.1:8085

sudo systemctl restart tor.service

Tor HiddenService:
http://afanfajsfafhwefih.onion/static.html

Reference:
•	https://jacobsalmela.com/2014/05/31/raspberry-pi-webcam-using-mjpg-streamer-over-internet/

•	https://github.com/jacksonliam/mjpg-streamer/issues/160

•	https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-phpmyadmin-on-ubuntu-12-04

•	https://gk2.sk/running-ssh-on-a-raspberry-pi-as-a-hidden-service-with-tor/

•	https://trac.torproject.org/projects/tor/wiki/doc/TorifyHOWTO/ssh

•	https://www.torproject.org/docs/tor-onion-service.html.en

