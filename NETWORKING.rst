**********************************************
Network Connections, stdin, stdout and  xinetd
**********************************************

Or "How the heck is ScarletDME communicating over the network"

`From https://www.thegeekdiary.com/understanding-etc-xinetd-d-directory-under-linux/ <https://www.thegeekdiary.com/understanding-etc-xinetd-d-directory-under-linux/>`__

"The xinetd daemon is a TCP wrapped super service which controls access to a subset of popular network services including FTP, IMAP, and telnet.
It also provides service-specific configuration options for access control, enhanced logging, binding, redirection, and resource utilization control.

When a client host attempts to connect to a network service controlled by xinetd ,
the super service receives the request and checks for any TCP wrappers access control rules.
If access is allowed, xinetd verifies that the connection is allowed under its own access rules for that service
and that the service is not consuming more than its allotted amount of resources or in breach of any defined rules.
It then starts an instance of the requested service and passes control of the connection to it.
Once the connection is established, xinetd does not interfere further with communication between the client host and the server."

xinetd takes the socket descriptor associated with TCP port X and maps it to the service's standard input stdin (fd "0")
and service's standard output stdout (fd "1").

In other words, assuming a custom TCP service written in C, data coming into port X is mapped to stdin of the service
and stdout of the service is mapped to data coming out of port Y. 

xinetd services read stdin and write to stdout, letting xinetd handle the gory details of TCP/IP, rather than keeping track of their own sockets

The "qmclient" and "qmserver" files in this directory should be copied to the
/etc/xinetd.d directory.  If you don't have this directory on your system,
you'll need to install the xinetd package for your system.

This directory also contains a file named "services".  The contents of this
file should be added to your /etc/services file.

Once you've done these things, you can restart xinetd via either 
"/sbin/service xinetd start" (or restart) or "/sbin/systemctl restart xinetd"
If your system supports neither of those methods, you'll need to consult the
documentation for your distribution.

sample "qmserver" file to be copied to the /etc/xinetd.d directory 

service qmserver::

 # This service supports connections to the ScarletDME
 # database server from telnet clients.
 # If your ScarletDME binary is located in a place other
 # than /usr/qmsys/bin, you'll need to update the server
 # entry below to reflect that new path.
 #
 # Port #4242 is normally used for this.
 service qmserver
 {
    protocol    = tcp
    flags       = REUSE
    socket_type = stream
    wait        = no
    groups      = yes
    user        = root
    group       = qmusers
    umask       = 002
    server      = /usr/qmsys/bin/qm
    server_args = -n
    log_on_failure  += USERID
    disable     = no
 }
 
sample "qmclient" file to be copied to the /etc/xinetd.d directory 

service qmclient::

 # This service supports connections to the ScarletDME
 # database server from qmclient clients.
 # If your ScarletDME binary is located in a place other
 # than /usr/qmsys/bin, you'll need to update the server
 # entry below to reflect that new path.
 #
 # Port #4243 is normally used for this.
 service qmclient
 {
    protocol = tcp
    flags = REUSE
    socket_type = stream
    wait = no
    groups = yes
    user = root
    group = qmusers
    umask = 002
    server = /usr/qmsys/bin/qm
    server_args = -n -q
    log_on_failure += USERID
    disable = no
 }

sample entries for your /etc/services file

/etc/services::

 # entries for your /etc/services file.
 qmserver    4242/tcp    # QMServer
 qmclient    4243/tcp    # QMClient


