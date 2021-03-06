#Minimalistic UPS Monitoring With Nut HowTo

Install nut on all machines that need information on the UPS state and the one the UPS is connected to (in case auf a USB or serial connection).

    apt-get install nut

Check the [hardware compatibility list](http://networkupstools.org/stable-hcl.html) for the needed driver (assuming usbhid-ups in the following).

## On The Machine The UPS is Connected to

### Setup UPS

Install udev rule:

    cp /lib/udev/rules.d/52-nut-usbups.rules /etc/udev/rules.d/
    udevadm control --reload-rules

Edit /etc/nut/nut.conf: It has only one non-comment line which should be:

    MODE=netserver

Then edit /etc/nut/ups.conf and add something like this:

    [myups]
        criver = usbhid-ups
        port = auto
        description = "My UPS / Location"

### Configure NUT Network Daemon (upsd)

In the first step /etc/nut/upsd.conf does not need any changes. But we need a user to connect to upsd and gather information about the UPS. Edit /etc/nut/upsd.users and add a users for local and remote monitoring:

    [locupsmon]
        password = topsecret
        upsmon master

    [remupsmon]
        password = topsecret
        upsmon slave

Now start the the UPS deamoon

    service nut-server start

### Check 

Check if your can connect to the UPS

    upsc myups@localhost

Now check if monitoring works. Add a MONITOR entry to /etc/nut/upsmon.conf

    MONITOR myups@localhost 1 locupsmon topsecret master

And start the monitoring service

    service nut-client start

Check syslog. You should find an entry like:

    upsd[12061]: User locupsmon@127.0.0.1 logged into UPS [myups]

### Switch From Localhost to Network Reachable

Add a LISTEN line to /etc/nut/upsd,conf (replace with your ip address; add more lines as needed):

    LISTEN 10.100.2.32 3493

and change the MONTIOR directive in /etc/nut/upsmon.conf accordingly

    MONITOR myups@10.100.2.32 1 locupsmon topsecret master

then restart client and server

    service nut-server restart
    service nut-client restart

### Setup SSL

Create a certificate in a NSS certificate database. Remeber the location of the certificate database (here /etc/nut/cert_db) and the passphrase (topsecret). Certificate CN and certificate nick in db should be host's ip address (10.100.2.32 in this example). [Check this HowTo](https://github.com/tweithoener/minimalistic-howtos/blob/master/certificates-and-ca-with-libnss3-tools.md)

Get owner and permissions straight:

    cd /etc/nut
	chown -R root:nut cert_db
	chmod 750 cert_db
	chmod 640 cert_db/*

In /etc/nut/upsd.conf add the following

    CERTPATH /etc/nut/cert_db
	CERTIDENT "10.100.2.32" "topsecret"

In /etc/nut/upsmon.conf add the following lines

    CERTPATH /etc/nut/cert_db
	CERTHOST "10.100.2.32" "10.100.2.32" 1 1
	CERTVERIFY 1
	FORCESSL 1

Then restart nut server and client

    service nut-server restart
    service nut-client restart

## On All Hosts That Need to Receive UPS Status Information

Install NUT

    apt-get install nut

In /etc/nut/nut.conf set MODE

    MODE=netclient

Get the upsmon.conf from the host with the UPS (the one configured before) ans playce it in /etc/nut.

Adjust owner and permission if needed

    chwon root:nut upsmon.conf
	chmod 640 upsmon.conf

Remove config files that are not needed

    cd /etc/nut
	rm ups.conf upsd.conf upsd.users

Change the MONITOR in /etc/nut/upsmon.conf directive to use the remote user

    MONITOR myups@10.100.2.32 1 remupsmon topsecret slave

Start the monitoring service

    service nut-client start

## Actions and Notifications

Config is in /etc/nut/upsmon.conf. Watch out for NOTIFYCMD, SHUTDOWNCMD, NOTIFYMSG and NOTIFYFLAG. 

Make sure /etc/init.d/nut-server and /etc/init.d/nut-client are run during boot (and shutdown).
