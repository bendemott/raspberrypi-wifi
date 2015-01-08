=================================================
Raspberry PI Access Point Setup and Configuration
=================================================

.. image:: http://i.imgur.com/PmZLggL.png
   :scale: 50
   :alt: captive-portal-page

If you follow the steps in this document, and read carefully, the end result will be a fully
functioning highly customizable raspberry-pi access point that you can use for a myriad of
entertaining and useful tasks.  

If you've ever wanted to see what a computer was doing while
connected to an access point, or see how different applications behave in slow or overcrowded
networks, this document, and the related commands in other files in this project can show you
how to have an entire WIFI test suite at your disposal.

----------------------
Build this doc as HTML
----------------------
#. Install docutils: ``sudo apt-get install python-docutils``
#. Build to HTML: ``rst2html howto.txt howto.html --stylesheet-path=docutils.css``

:Note: If ``rst2html`` isn't present try: ``rst2html.py``

------------
IP Addresses
------------
| In this tutorial I'm going to assume you will be creating a new network and subnet in the ``192.168.1.xxx`` range.
|
| I'm also going to assume you already have an existing **LAN** network that is in the
| ``10.0.10.xxx`` range.
| 
| If your LAN is using ``192.168.1.1`` and you want the Raspberry PI to use
| ``10.x.x.x`` range, you should switch around the commands as you see fit.

:Note: I will eventually add some scripts to help build the configs automatically.

-------------------
Installing Raspbian
-------------------
**Raspbian** is the default and best linux distribution currently available to use with your
raspberry pi.  Our raspberry pi came with a bootable memory card with a program called **NOOBS**
on it.  You can use NOOBS to install Raspbian (it gives you several other options).

""""""""""
From Noobs
""""""""""

    - Plug in your keyboard and monitor (hdmi)
    - Plug in the USB power cable
    - The Raspberry PI will begin booting (you'll see lights on the board lit)
      A list of the different operating systems will appear. Tick the box next to Raspbian
      and select ``install``.
    - The install process wil begin (this will take awhile)
    - ``raspi-config`` is automatically started when install finishes.  You can set the region
      time, date, etc from this menu.

""""""""""""""""""""""""""""""""""""   
Restoring/Installing from Disk Image
""""""""""""""""""""""""""""""""""""

    - Insert the memory card into a memory card reader and connect it to your
      linux desktop.
    - Use ``dd`` command line tool to copy your disk image to the raspberry pi.
    - You're done.

------------------------------
Logging In to the Raspberry PI
------------------------------
When connected directly to the raspberry pi or over ssh you can use default credentials.

`Note:` you won't see any characters appear when you're typing (this is a security feature of linux)

**Default Credentials**::
  
    username: pi
    password: raspberry
    
**Login**:

    ``ssh pi@10.0.10.48``

--------------------------
Installing and Configuring
--------------------------

"""""""
Configs
"""""""
Configuration files we'll be creating or editing.

Copies of some of these are located in the **configs** directory within the github
repository containing this code based on exactly the configurations detailed here.

- /etc/sysctl.conf
- /etc/default/hostapd
- /etc/dnsmasq.conf
- /etc/hostapd/hostapd.conf
- /etc/network/interfaces

"""""""""""""""""""""""""""""
Setup the **wlan0** interface
"""""""""""""""""""""""""""""
We need the wireless hardware interface setup and configured in such a way that it can act as
an access point.  To do this we'll need to do a few things.

#. Edit ``/etc/network/interfaces``
#. Comment out these lines::
    
    #allow-hotplug wlan0
    #wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
    #iface default inet dhcp
    
#. Add these lines (editing the ``iface wlan0`` interface if it's already there)::

    iface wlan0 inet static
      address 192.168.1.1
      netmask 255.255.255.0

"""""""""""""""""""
Install **hostapd**
"""""""""""""""""""
**hostapd** will act as the actual access point software.  It will manage the wireless device and
broadcast the wireless network.

**WIFI Adapter Note:**

    Many Raspberry PI use Wifi Adapter Realtek 8188CU (`same chip used by the Edimax EW-7811Un`)
    This ADAPTER is Not supported and will not run in AP mode!  
    
    There are mentions of a patch to ``hostapd`` that allows this chip to work, but it was unsuccessful for me.
    
#. | **Verify AP Support:**
   | Verify by ``sudo apt-get install iw`` and executing ``iw list``.  
   | If you see ``nl80211 not found.`` your device doesn't support AP mode.
   | At SpotOn we purchased `Panda Mid-Range Wireless N USB Adapter's` - these worked great.
#. ``sudo apt-get install hostapd``
#. | Edit the file ``/etc/default/hostapd`` edit and uncomment the existing line in the file so
     you end up with...
   | ``DAEMON_CONF="/etc/hostapd/hostapd.conf"``
#. Create/Edit the file ``/etc/hostapd/hostapd.conf``
#. Change/Add the line ``driver=nl80211`` to the hostapd.conf file.
#. | Add the following configuration options to hostapd.conf
   | change ``ssid`` to a reasonable name
   | and ``wpa-passphrase`` to something unique.
   | Change ``wlan0`` to the name of your wireless interface.
   
   ::
    
        driver=nl80211
        interface=wlan0
        country_code=US
        #ctrl_interface=eth0
        #ctrl_interface_group=0
        ssid=rpi1-at-10-0-10-48
        hw_mode=g
        channel=1
        ignore_broadcast_ssid=0
        wpa=2
        wpa_passphrase=password
        wpa_key_mgmt=WPA-PSK
        wpa_pairwise=TKIP
        rsn_pairwise=CCMP
        beacon_int=100
        auth_algs=3
        macaddr_acl=0
        wmm_enabled=1
        eap_reauth_period=360000000
        logger_stdout=1
        logger_stdout_level=0
        logger_syslog=-1
        logger_syslog_level=0

#. | Start **hostapd**
   | ``sudo service hostapd start`` ... (or)
   | ``sudo hostapd /etc/hostapd/hostapd.conf`` to run interactively (view errors)
   
#. | **optional step**
   | Add the line ``ifconfig wlan0 192.168.1.1`` to the file
   | ``/etc/init.d/hostapd`` immediately before the **exit 0** at the bottom.  
   |
   | When ``hostapd`` is restarted, the IP address on the WIFI (`wlan0`) interface is lost.
   | Now the command ``service hostapd restart`` will function properly.

"""""""""""""""""""
Install **dnsmasq**
"""""""""""""""""""

    #. ``sudo apt-get install dnsmasq``
    #. | edit the file ``/etc/dnsmasq.conf``
       | uncomment and set ``interface=wlan0``
       | uncomment and set ``dhcp-range=192.168.1.50,192.168.1.150,255.255.255.0,12h``
       | uncomment and set ``dhcp-authoritative``
    
""""""""""""""""""""""""
Setup connection sharing
""""""""""""""""""""""""
**Assumptions:**

    - You have a wired connection to a private network behind a router as interface ``eth0``.
    - You have a wireless connection configured as ``wlan0``.

**IpTables Note:**

    - You can check iptablets and the current ruleset with the command ``sudo iptables -nvL``
    - You can flush and reset iptables (`to defaults`) with these commands::

        iptables -F
        iptables -X
        iptables -t nat -F
        iptables -t nat -X
        iptables -t mangle -F
        iptables -t mangle -X
        iptables -t raw -F
        iptables -t raw -X
        iptables -t security -F
        iptables -t security -X
        iptables -P INPUT ACCEPT
        iptables -P FORWARD ACCEPT
        iptables -P OUTPUT ACCEPT

#. Enable NETWORK ADDRESS TRANSLATION (NAT) to share the internet on eth0 with wlan0...::

    sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"
    
#. Persist NAT after boot - Edit ``/etc/sysctl.conf`` adding this line at the bottom::

    net.ipv4.ip_forward=1

#. Enable NAT/forwarding in the kernel::

    sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
    sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
    sudo iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
    
#. Persist the changes to iptables::

    sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
    
#. | Now edit the file ``/etc/network/interfaces`` and add the following line at the bottom
   | This will cause iptables to be restored from config when the interface comes up::
   
    up iptables-restore < /etc/iptables.ipv4.nat
    
""""""""""""""""""""""
Services Configuration
""""""""""""""""""""""
We want ``hostapd``, and ``dnsmasq`` to start with the device by default.::

    sudo update-rc.d hostapd enable
    sudo update-rc.d dnsmasq enable

----------------
Networking Tests
----------------

- See the file ``tc.rst`` for poor network quality simulation options (`packet loss, out-of-order, etc`) 
- See the file ``iptables.rst`` for network failure simulation commands.
- See the file ``dnsredirection.rst`` for controlling how DNS requests for specific domains are handled.

----------
References
----------

**The most Relevant tutorial specifically for RPI and using dnsmasq and hostap**

    - http://sirlagz.net/2012/08/09/how-to-use-the-raspberry-pi-as-a-wireless-access-pointrouter-part-1/
    - http://sirlagz.net/2012/08/10/how-to-use-the-raspberry-pi-as-a-wireless-access-pointrouter-part-2/
    - http://sirlagz.net/2012/08/11/how-to-use-the-raspberry-pi-as-a-wireless-access-pointrouter-part-3/

**Other stuff**

    - http://elinux.org/RPI-Wireless-Hotspot
    - http://www.daveconroy.com/turn-your-raspberry-pi-into-a-wifi-hotspot-with-edimax-nano-usb-ew-7811un-rtl8188cus-chipset/
    - http://blog.sip2serve.com/post/48420162196/howto-setup-rtl8188cus-on-rpi-as-an-access-point
    - https://github.com/previ/coderbot/wiki/Realtek-8188-Wi-Fi-adapter
    - http://www.raspberrypi.org/forums/viewtopic.php?p=462982#p462982
    - http://www.andybev.com/index.php/Using_iptables_and_PHP_to_create_a_captive_portal
    - http://ftp.netbsd.org/pub/NetBSD/NetBSD-current/src/external/bsd/wpa/dist/hostapd/hostapd.conf

**forwarding between networks**

    - http://serverfault.com/questions/267580/linux-routing-traffic-between-two-networks-with-iptables

**The best article on configuring NAT with dnsmasq and access point.**

    - https://wiki.archlinux.org/index.php/Software_access_point
