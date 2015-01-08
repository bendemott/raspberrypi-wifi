Useful Commands for Interacting with the Raspberry PI Access Point
==================================================================
``raspi-config`` - This is how raspbian does basic config.

``lsusb`` - This lists USB hardware connected to the device.  You can find out your Wireless 
            adapter type this way.
            
``iw list`` - This command lists the available abilities.  Under `supported interface modes` you
              can see if the wireless interface supports acting as an access point (`AP`).
              You must ``apt-get install iw`` before this works.
              
``iwconfig`` - This will display the Mode the Access Point is running in (assuming USB).
                    It also displays the associated interface with the hardware device.
                    
``iwconfig wlan0 txpower 27`` This is the US legal maximum of 27 dBm (500 milliwatts)

``iwlist wlan0 scan`` List all networks and channels in wireless range.

``ifconfig wlan0 down``, ``ifconfig wlan0 up`` - Restart the wlan 0 adapter.

``tail -f /var/log/syslog -n 1000 | sed -u "s/#012/\n\t/gp" | ccze --raw-ansi`` - Colored logging

``sudo ifconfig wlan0 192.168.1.1`` Issue this command after restarting hostapd ``service hostapd restart``
