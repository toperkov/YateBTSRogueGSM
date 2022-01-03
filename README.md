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
$ wget https://github.com/toperkov/YateBTSRogueGSM/raw/main/yate-rc.tar
$ tar xvf yate-rc.tar
$ mv yate /usr/src
$ mv yate-bts /usr/src
$ mv *.rbf /usr/share/nuand/bladeRF
$ apt install autoconf gcc g++ make
```

Step 5: Compile Yate

```diff
--- modules/openssl.cpp
+++ modules/openssl.cpp
@@ -36,6 +36,10 @@
 #include <openssl/des.h>
 #endif
 
+#if OPENSSL_VERSION_NUMBER >= 0x10100000L
+#include <openssl/modes.h>
+#endif
+
 using namespace TelEngine;
 namespace { // anonymous
 
@@ -644,6 +648,17 @@
 	inpData = outData;
     unsigned int num = 0;
     unsigned char eCountBuf[AES_BLOCK_SIZE];
+#if OPENSSL_VERSION_NUMBER >= 0x10100000L
+    CRYPTO_ctr128_encrypt(
+	(const unsigned char*)inpData,
+	(unsigned char*)outData,
+	len,
+	m_key,
+	m_initVector,
+	eCountBuf,
+	&num,
+	(block128_f)AES_encrypt);
+#else
     AES_ctr128_encrypt(
 	(const unsigned char*)inpData,
 	(unsigned char*)outData,
@@ -652,6 +667,7 @@
 	m_initVector,
 	eCountBuf,
 	&num);
+#endif
     return true;
 }
```

```Linux
$ cd /usr/src/yate
$ ./autogen.sh
$ ./configure —-prefix=/usr/local
$ make
$ make install
$ ldconfig
```

Step 6: Compile YateBTS

```diff
#This patch works for gcc6, gcc7, and is from http://yate.null.ro/mantis/view.php?id=416
--- a/mbts/GPRS/MSInfo.cpp
+++ b/mbts/GPRS/MSInfo.cpp
@@ -638,7 +638,7 @@
         if (msPCHDowns.size() > 1) {
             std::ostringstream os;
             msDumpChannels(os);
-            GPRSLOG(INFO,GPRS_MSG|GPRS_CHECK_OK) << "Multislot assignment for "<<this<<os;
+            GPRSLOG(INFO,GPRS_MSG|GPRS_CHECK_OK) << "Multislot assignment for "<<this<<(!os.fail());
         }
 
 	} else {
--- a/mbts/SGSNGGSN/Sgsn.cpp
+++ b/mbts/SGSNGGSN/Sgsn.cpp
@@ -149,7 +149,7 @@
 	clearConn(GprsConnNone,SigConnLost);
 	std::ostringstream ss;
 	sgsnInfoDump(this,ss);
-	SGSNLOGF(INFO,GPRS_OK|GPRS_MSG,"SGSN","Removing SgsnInfo:"<<ss);
+	SGSNLOGF(INFO,GPRS_OK|GPRS_MSG,"SGSN","Removing SgsnInfo:"<<(!ss.fail()));
 	sSgsnInfoList.remove(this);
 	GmmInfo *gmm = getGmm();
 	if (gmm && (gmm->getSI() == this)) {
@@ -252,7 +252,7 @@
 {
 	std::ostringstream ss;
 	gmmInfoDump(gmm,ss,0);
-	SGSNLOGF(INFO,GPRS_OK|GPRS_MSG,"SGSN","Removing gmm:"<<ss);
+	SGSNLOGF(INFO,GPRS_OK|GPRS_MSG,"SGSN","Removing gmm:"<<(!ss.fail()));
 	SgsnInfo *si;
 	RN_FOR_ALL(SgsnInfoList_t,sSgsnInfoList,si) {
 		// The second test here should be redundant.
```

```Linux
$ cd /usr/src/yate-bts
$ ./autogen.sh
$ ./configure —-prefix=/usr/local
$ make
$ make install
$ ldconfig
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
