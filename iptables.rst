-----------------------------
Testing Network Functionality
-----------------------------
Linux **iptables** can be very useful in simulating various types of network failures to test
scenarios within applications.

Connect to the Raspberry PI Access Point for testing.
Enter the commands found below through ssh, to test each particular type of functionality.

If you are in a "development" mode and are using alternate ports, please change port "80" or "443"
in the examples below to the test port you are actually using.

**Raspberry PI Credentials**::

    SSH
        user: pi
        pass: raspberry
        
    WIFI
        ssid: rpi1-at-10.0.10.48
        pass: password
    

**IpTables Notes**:

    - ``sudo iptables-restore < /etc/iptables.ipv4.nat`` - **Restores iptables** to startup config
      If you want to fully reset iptables (`instead of restoring to boot-time settings`) you can issue:
      ``sudo iptables -F`` `this breaks the access point NAT forwarding`
    - ``sudo netstat -tunp|grep 192.168.1.1`` - **Reset** connections to the Raspberry PI
    - ``sudo iptables -nvL`` - **Lists** all iptables rules
    - to **REMOVE** any rule after you are done with a particular test, replace ``-A`` with ``-D``
      in the command.
    - Any **REJECT** command can be replaced with **DROP** this will simulate timeout-like
      conditions instead of a ``ConnectionRefused`` event.


**Test HOST Failure**::
    
    # Test a simulated communication failure somewhere in the network link
    # between a SERVER and the peer/client.
    # replace the IP-ADDRESS with the server you are pointing to.
    # you can use ``nslookup yourhost.com`` to find the ip address of a server.
    
    sudo iptables -I FORWARD -p tcp --destination-port 443 -j REJECT -d 204.11.50.136
    
**Test HTTP Networking Failure (no-route)**::

    # block port 80 and 443 all interfaces.
    # This simulates a complete network failure, no requests will
    # succeed.
    sudo iptables -I FORWARD -m multiport -p tcp --dports 80,443 -j REJECT
    
**Test HTTP Networking Failure (timeout)**::

    # instead of rejecting packets, ignore all http/https packets destined for the internet
    # this simulates a timeout.
    sudo iptables -I FORWARD -m multiport -p tcp --dports 80,443 -j DROP
    
**Test HTTPS Blocked**::

    # This simulates all HTTPS (secure) traffic being blocked.
    # This test is useful to test services, clients, or api's that should negotiate between
    # http and https.
    sudo iptables -I FORWARD -p tcp --dport 443 -j REJECT

**Test DNS (connection refused)**::

    # before testing any DNS failure, you must clear the browser cache,
    # and close any open programs on the client, and ensure there are no
    # lingering http connections by restarting networking on the client.
    
    # Reject all outgoing DNS requests.
    sudo iptables -I INPUT -p udp --sport 53 -j REJECT

**Test DNS Failure (timeout)**::

    # Simulates DNS requests that timeout. 
    sudo iptables -I FORWARD -p udp --sport 53 -j DROP
    
    
**Test Network Captive Portal**::

    # Test all requests succeeding, but not returning the content that was in
    # fact requested.
    sudo apt-get install python-twisted python-twisted-web
    sudo twistd web -n -p 80 --path /usr/share/doc/debian-reference-common/html
    sudo iptables -t nat -I PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.1:80
    sudo iptables -t nat -I PREROUTING -p tcp --dport 443 -j DNAT --to-destination  192.168.1.1:80
    sudo iptables -t nat -I POSTROUTING -j MASQUERADE
    
    # -- CLEANUP ---------------
    # to kill the server you just started:
    sudo kill `sudo cat twistd.pid`
    
**Test HTTP Captive Portal**::
    
    # First install the captive portal from GITHUB
    sudo apt-get install python-pip
    sudo pip install git+https://github.com/bendemott/captiveportal.git
    
    # This tests a redirection (302) based captive portal.  Clients detect this by
    # listening for redirects from multiple requests to multiple domains.
    # Note that if your client is using an alternative port or proxy (such as 8080) the captive
    # portal won't work because communication to the server will bypass the default http port.
    
    # start the portal, and the redirection of traffic.
    sudo captiveportal start
    
    # to reset the captive portal clients list
    sudo captiveportal reset
    
    # to stop the captive portal
    sudo captiveportal stop
    
**Test tablet PROXY**::

    # Configure a proxy inside of the tablet (configure to use HTTP proxy)
    # and cause a connection failure.
    - Connect to WIFI network (e.g. 'AirportXtreme')
    - Settings->WIFI
    - Long tap on connected network's name (e.g. on 'SpotOn')
    - Modify network config-> Show advanced options
    - Set proxy settings (set to arbitrary host/non-existant)
    # Block all connections to force the proxy settings to be evaluated.
    sudo iptables -I FORWARD -m multiport -p tcp --dports 80,443 -j REJECT

**Network Failure, Gateway Reachable**::

    sudo apt-get install python-twisted python-twisted-web
    sudo twistd web -n -p 80 --path /usr/share/doc/debian-reference-common/html
    # Default Policy to REJECT all forwarded traffic
    sudo iptables -P FORWARD REJECT
    # add an exception for our IP address
    sudo iptables -I FORWARD --destination 192.168.1.1 -j ACCEPT
    
    # to cleanup afterwards (kill the web server)
    sudo kill `sudo cat twistd.pid`
    

**Network Failure, Gateway Unreachable**::
    
    # Simulate the gateway being unreachable by HTTP/HTTPS connections and ping.
    # To simulate the gateway being unreachable, we have to block outgoing http/https
    # then block http traffic to us directly.
    # then block icmp traffic to us directly.


    sudo iptables -I FORWARD -m multiport -p tcp --dports 80,443 -j REJECT
    sudo iptables -I INPUT -m multiport -p tcp --dports 80,443 -j REJECT
    sudo iptables -I INPUT -p icmp --icmp-type echo-request -j DROP
    sudo iptables -I OUTPUT -p icmp --icmp-type echo-reply -j DROP
    sudo iptables -I INPUT -p tcp --dport 7 -j DROP  # for Android devices.
