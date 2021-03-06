# Sshguard

Sshguard monitors log files of services running on your machine and when it detects that someone is doing something bad to a service it adds a firewall rule that blocks the IP address of who abused it. After some time, it releases the blocking.

Installation

	emerge app-admin/sshguard

Add these required iptables rules

	# Create a new chain in which sshguard will append blocking rules
	iptables -N sshguard

	# Block any traffic from abusers
	iptables -A INPUT -j sshguard


Sample `/etc/conf.d/sshguard` configuration file

	# Initial (empty) options.
	SSHGUARD_OPTS="-p 3600 -s 3600 -a 20"

	# Files to monitor
	#   -l <source>
	SSHGUARD_OPTS="${SSHGUARD_OPTS} -l /var/log/messages"

	# White listing
	#   -w <addr/host/block/file>
	SSHGUARD_OPTS="${SSHGUARD_OPTS} -w 192.168.0.0/24"

	# Define how long in milliseconds start-stop-daemon waits to check that
	# sshguard is still running before calling success or failure.
	# Values lower than the default of 999 are probably not useful.
	SSHGUARD_WAIT="999"

Edit `/etc/syslog-ng/syslog-ng.conf` to instruct syslog-ng to pass service logs to sshguard


	# sshguard
	filter f_sshguard { facility(auth, authpriv) and not program("sshguard"); };
	destination sshguard {
		program("/usr/sbin/sshguard"
			template("$DATE $FULLHOST $MSGHDR$MESSAGE\n")
		);
	};
	log { source(src); filter(f_sshguard); destination(sshguard); };

Restart syslog-ng and sshd

	/etc/init.d/syslog-ng restart
	/etc/init.d/sshd restart

Add to the rc levels to ensure it starts at every boot

	rc-update add sshguard default

Start the service

	/etc/init.d/sshguard start
