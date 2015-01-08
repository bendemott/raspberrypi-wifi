Unable to Connect
=================
Sometimes after restarting hostapd, client devices can't obtain IP-Addresses.
This seems to be correlated to this message in syslog::

    dnsmasq-dhcp[2699]: DHCP packet received on wlan0 which has no address

I found this blog post which talked about a hack/fix:

    I “fixed” the problem with the ugly hack of editing the init script for hostapd, 
    and inserting the command ifconfig wlan0 192.168.1.1 such that it runs after the hostapd daemon.
    
In our case issuing this command after ``service hostapd restart`` works

    sudo ifconfig wlan0 192.168.1.1
