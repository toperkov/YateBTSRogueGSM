# YateBTSRogueSGM
Rogue GSM Base Station based on YateGSM

Step 0: Enter root mode with:

```Linux
$ sudo su
```


Step 1: Update/upgrade your fresh installation of Ubuntu. In this tutorial, I’m using Ubuntu 20.04 LTS.

```Linux
$ apt update ; apt upgrade
```

Step 2: Add BladeRF PPA and install BladeRF tools and libbladeRF

```Linux
$ add-apt-repository ppa:bladerf/bladerf
$ apt-get update
$ apt-get install bladerf
$ apt-get install libbladerf-dev
```

Step 3: Add user/group permissions for non-root user

```Linux
$ addgroup yate
$ usermod -a -G yate xxxxxx-your-username-here-xxxxxx
$ apt install libusb-1.0-0-dev
```

Step 4: Download the custom Yate distro created by Nuand

```Linux
$ wget https://www.nuand.com/downloads/yate-rc.tar
$ tar xvf yate-rc.zip
$ mv yate /usr/src
$ mv yatebts /usr/src
$ mv *.rbf /usr/share/nuand/bladeRF
$ apt install autoconf gcc g++ make
```

Step 5: Compile Yate

```Linux
$ cd /usr/src/yate
$ ./autogen.sh ; ./configure —-prefix=/usr/local ; make ; make install-noapi ; ldconfig
```

Step 6: Compile YateBTS

```Linux
$ cd /usr/src/yatebts
$ ./autogen.sh
$ ./configure —-prefix=/usr/local
$ make
$ sudo make install
$ sudo ldconfig
```

Step 7: Set permissions

```Linux
$ touch /usr/local/etc/yate/snmp_data.conf /usr/local/etc/yate/tmsidata.conf

$ chown root:yate /usr/local/etc/yate/*.conf
$ chmod g+w /usr/local/etc/yate/*.conf
```

Step 8: Set transceiver scheduling

```Linux
$ vi /usr/local/etc/yate/ybts.conf
```

# Add the values below to the ybts.conf file

```Linux
radio_read_priority=highest
radio_send_priority=high
```

Step 9: Install Apache2 and PHP

```Linux
$ apt install apache2
$ add-apt-repository ppa:ondrej/php
$ apt update
$ apt install php5.6
```

Step 10: Install Network-in-a-PC


```Linux
$ cd /var/www/html
$ ln -s /usr/local/etc/yate/nipc_web nipc
$ chmod -R a+rw /usr/local/etc/yate
$ /etc/init.d/apache2 start
```

Step 11: Connect to NIPC with your web browser and configure MCC, MNC, and Band for your BTS. NOTE: To determine what values to use here, select a wireless network to act as a decoy (E.g. AT&T Wireless). If your mobile phone is connected to the wireless network you want to imitate a BTS for, you can place your phone into field test mode. Here is the code you need to dial for an iPhone and Android:

1. Push the call button to make a phone call
2. Dial *3001#12345#*
3. Push the Call button
4. Push Serving Cell Info

Step 12: Plug in the BladeRF to the USB cable and laptop and soft load the FPGA

```Linux
$ bladeRF-cli -l /usr/src/Nuand/bladeRF/hostedxA9.rbf (or whatever FPGA file matches your board)
```

Step 13: Start Yate

```Linux
$ sudo yate -v
```
