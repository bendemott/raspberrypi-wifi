The ``tc`` command along with ``netem`` performs network emulation and shaping.

http://www.linuxfoundation.org/collaborate/workgroups/networking/netem

Install
=======
sudo apt-get install netemul

Notice
======
- be careful not to traffic shape the eth0 connection you are connected through.
  It is possible to disable or traffic shape your own SSH connection.
- All delays are limited to the clock resolution of the kernel.

Examples
========

::

	-- RESET OPTIONS -------------------------------------------
	tc qdisc del dev wlan0 root

	-- NETWORK DELAY -------------------------------------------
	# fixed delay to all packets on wlan0
	tc qdisc add dev wlan0 root netem delay 100ms

	# Add random variation
	tc qdisc change dev wlan0 root netem delay 100ms 10ms

	# Add +/- 10ms Network delay, this way it isn't truly random.
	tc qdisc change dev wlan0 root netem delay 100ms 10ms 25%


	-- COMPLEX DELAY DISTRIBUTION -----------------------------
	# create non-uniform network delays based on a predefined distribution.

	# normal
	tc qdisc change dev wlan0 root netem delay 100ms 20ms distribution normal

	# Or... pareto ... (normal, pareto, paretonormal) see /usr/lib/tc
	tc qdisc change dev wlan0 root netem delay 100ms 20ms distribution pareto


	-- NETWORK PACKET LOSS ------------------------------------
	# This causes 1/10th of a percent (i.e. 1 out of 1000) packets to be randomly dropped.
	tc qdisc change dev wlan0 root netem loss 0.1%
	
	# This will cause 0.3% of packets to be lost, 
	# and each successive probability depends by a quarter on the last one.
	tc qdisc change dev wlan0 root netem loss 0.3% 25%

	-- PACKET CORRUPTION -------------------------------------
	tc qdisc change dev wlan0 root netem corrupt 0.1%

