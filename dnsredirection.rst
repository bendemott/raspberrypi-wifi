""""""""""""""""""""""""
Redirecting DNS Requests
""""""""""""""""""""""""
One of the features of using ``dnsmasq`` and the **Raspberry Pi** for our networking needs is the
ability to send a clients request for ``google.com`` somewhere else entirely.  This can be useful
for examining how applications, or networking services function, or for your own development
purposes.


----------------
Address Argument
----------------

    we need to add an ``address=`` argument to the **/etc/dnsmasq.conf** file.
    
    Find this section in the ``dnsmasq.conf`` file::
    
        # Add domains which you want to force to an IP address here.
        # The example below send any host in double-click.net to a local
        # web-server.
        #address=/double-click.net/127.0.0.1
        
    Edit the file, adding a line below. In this example I redirect
    **google.com** to ``10.0.10.15``.::
    
        address=/google.com/10.0.10.15
    
    If my local computer had an ip address of
    ``10.0.10.15`` on the LOCAL NETWORK, all network requests by any client 
    connected to the ACCESS POINT would be sent to my computer.
