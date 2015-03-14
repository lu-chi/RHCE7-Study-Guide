# RHCE7-Study-Guide
My Study Guide for RHCE7



## Controlling Services and Daemons - Complete

###Concepts
#### Unit Types

* Service Units: have a .service extension and represent system services. This type of unit is used to start frequently accessed daemons, such as a web server.

* Socket Units: have a .socket extension and represent interprocess communication (IPC) sockets. Control of the socket will be passed to a daemon or newly started service when a client connection is made. Socket units are used to delay the start of a service at boot time and to start less frequently used servies on demand. These are similarin principle to services which use the xinetd superserver to start on demand.

* Path Units: have a .path extension and are used to delay the activation of a service until a specific file system change occurs. This is commonly used for services which use spool directories, such as a printing system.

#### Targets
A target is a set of systemd units that should be started to reach a desired state. 
##### Important targets
 
 * graphical.target: System supports multiple users, graphical and text-based logins.

 * multi-user.target: System supports multiple users, text-based logins only. 

 * rescue.target: sulogin prompt, basic system initialization completed.

 * emergency.target: sulogin prompt, initramfs pivot complete and system root mounted on / read-only.

 ##### Selecting a different target at boot time.
 Simply interrupt the boot loader menu, select the entry to be started, edit it by pressing 'e' and append "systemd.unit=<DESIRED_UNIT>" to the line starting with linux16

#### Recovering the root password

* Reboot the System, enter edit mode for the proper entry in the bootloader. 

* Append rd.break to the end of the line that starts with linux16. Restart with Ctrl+X

* Remount /sysroot as read-write. 

```bash
switch_root:/# mount -oremount,rw /sysroot
```

* Switch into a chroot jail, where /sysroot is treated as the root of the file system tree.

```bash
chroot /sysroot
```

* Set a new root password

* Make sure that all unlabeled files (including /etc/shadow at this point) get relabeled during boot. 

```bash
touch /.autorelabel
```

* Type exit twice to continue booting.

### Commands

#### Common systemctl commands

```bash
systemctl status <UNIT>
```
```bash
systemctl stop <UNIT>
```
```bash
systemctl start <UNIT>
```
```bash
systemctl restart <UNIT>
```
```bash
systemctl reload <UNIT>
```
```bash
systemctl mask <UNIT>
```
```bash
systemctl unmask <UNIT>
```
```bash
systemctl enable <UNIT>
```
```bash
systemctl disable <UNIT>
```
```bash
systemctl list-dependencies <UNIT>
```
Change the system target during runtime: 
```bash
systemctl isolate <TARGET>
```
Get default system target
```bash
systemctl get-default
```
Set the default target (Not for runtime)
```bash
systemctl set-default <DESIRED_TARGET>
```

## Managing IPv6 Networking
###Concepts
####Modifying the system hostname
A static host name may be specified in the /etc/hostname file. The hostnamectl command is used to modify this file and may be used to view of the systems FQDN. 



###Commands
List names of available connections:
```bash
nmcli con show
```
Adding a new connection profile
```bash
nmcli con add con-name <NAME> type <TYPE> ifname <INTERFACE_NAME>
```
Modifying existing connections
```bash
nmcli con mod <con-name> ..
```
Deleting a network connection
```bash
nmcli con del <con-name>
```
Deactivate and disconnect the current connection on the network interface dev
```bash
nmcli dev dis <dev-name>
```

## Configuring Link Aggregation and Bridging - Complete
###Network Teaming
Software, called runners, implement load balancing and active-backup logic, such as roundrobin. The following runners are avaialble:

* broadcast: a simple runner which transmits each packet from all ports.

* roundrobin: a simple runner which transmits packets in a round-robin fashin from each of the ports

* activebackup: This is a failover runner which watches for link changes and slects an active port for data transfers. 

* loadbalance: This runner monitors traffic and uses a hash function to try to reach a perfect balance when selecting ports for packet transmission.

* lacp: implements the 802.3ad Link Aggregation Control Protocol. 

####Configuring Network Teams:
Create the team interface:
```bash
nmcli con add type team con-name team0 ifname team0 config '{"runner":{"name":"loadbalance"}}'
```
Determine the IPv4/IPv6 attributes of the team interface
```bash
nmcli con mod team0 ipv4.address 1.2.3.4/24
nmcli con mod team0 ipv4.method manual
```
Assign the port interfaces. The connection name can be explicitly specified, or it will be team-slave-IFACE by default
```bash
nmcli con add type team-slave ifname eth1 master team0
nmcli con add type team-slave ifname eth2 master team0
```
Bring the team and port interfaces up/down
```bash
nmcli con up team0
nmcli dev dis eth2 
```
The teamdctl can be used to display the teams state. 
```bash
teamdctl team0 state
```
####Setting and adjusting team configuration
```bash
nmcli con mod IFACE team.config JSON-configuration-file-or-string
```
NOTE: Any changes made do not go into effect until the next time the team interface is brought up

#####Link watch setting
The link watch setting determines how the link state of the port interfaces are monitored. The default uses functionality similar to the ethtool command to check the link of each interface. Another way to check link state is to periodically use an ARP ping packet to check for remote connectivity. For Example:
```bash
"link_watch": {
	"name": "arp_ping",
	"interval": 100,
	"missed_max": 30,
	"source_host": "192.168.23.2",
	"target_host": "192.168.23.1"
},
```
####Troubleshooting network teams
```bash
teamdctl team0 config dump
```

###Configuring Software Bridges
```bash
nmcli con add type bridge con-name br0 ifname br0
nmcli con add type bridge-slave con-name br0-port1 ifname eth1 master br0
nmcli con add type bridge-slave con-anme br0-port2 ifname eth2 master br0
```
NOTE: NetworkManager can only attach Ethernet interfaces to a bridge. It does not support aggregate interfaces, such as a teamed or bonded interface. These must be configured by manipulating the configuration files in /etc/sysconfig/network-scripts.

####Notes on adding a teamed interface to a bridge
Disable the teamed interface in network manager, disable network manager
```bash
nmcli dev dis team0
systemctl stop NetworkManager
systemctl disable NetworkManager
```
Add the BRIDGE entry to the teamed interface
```bash
BRIDGE=brteam0
```
Delete the IP configurations from the configurations of the team port interfaces.

Create a new interace configuration file for the bridge. Define the configuration information in that file:
```bash
DEVICE=brteam0
ONBOOT=yes
TYPE=Bridge
IPADDR0=192.168.0.100
PREFIX0=24
```
Reset the network
```bash
systemctl restart network
```



## Network Port-Security
###Firewalld
Some important commands:
```bash
firewall-cmd --set-default-zone=dmz
firewall-cmd --permanent ...
firewall-cmd --reload
```
####Rich rules
Examples:
```bash
firewall-cmd --permanent --zone=classroom --add-rich-rule='rule family=ipv4 source address=192.168.0.11/32 reject'
firewall-cmd --add-rick-rule='rule service name=ftp limit value=2/m accept'
firewall-cmd --permanent --add-rich-rule='rule protocol value=esp drop'
```

Logging with rich rules:
```bash
firewall-cmd --permanent --zone=work --add-rich-rule='rule service name="ssh" log prefix="ssh " level="notice" limit value="3\m" accept'
```

For Help, there are examples at the bottom:
```bash
man firewalld.richlanguage
```
###Masquerading
```bash
firewall-cmd --permanent --zone=<ZONE> --add-masquerade
```
###Port Forwarding
```bash
firewall-cmd --permanent --zone=<ZONE> --add-forward-port=port=<PORTNUMBER>:proto=<PROTO>[:toport=<PORTNUMBER>][:toaddr=<IPADDR>]
```
###Managing SELinux Port Labeling
Whenever an administrator decides to run a service on a nonstandard port, there is a high chance that SeLinux port labels will need to be updated. 

List Port Labels:
```bash
semanage port -l
```

Managing port labels. To add a port to an existing port label:
```bash
semanage port -a -t port_label -p tcp|udp PORTNUMBER
semanage port -a -t gopher_port_t -p tcp 71
```

Remove port labels
```bash
semanage port -d -t gopher_port_t -p tcp 71
```

Modifying port binding, this will modify port 71/tcp from gopher_port_t to httpd_port_t:
```bash
semanage port -m -t httpd_port_t -p tcp 71
```


## Managing DNS for Servers
###DNS Concepts
Resource Records:

* A (Ipv4) records - An A record maps a host name to an IPv4 address

* AAAA (Ipv6 address) records - An AAAA resource record maps a host name to an IPv6 address.

* CNAME(canonical name) records - A CNAME resource record aliases one name to another name (the canonical name), which should have A or AAAA records.

* PTR(pointer) records - A PTR record maps IPv4 or IPV6 addresses to a host name. They are used for reverse DNS resolution

* NS(name server) records - An NS record maps a domain name to a DNS name server which is authoritive for its DNS zone

* SOA(start of authority) records - an SOA record provides information about how a DNS zone works. 

* MX (mail exchange) records - An MX record maps a domain name to a mail exchange which will accept email for that name.

* TXT(text) records - A TXT record is used to map a name to arbitrary human-readable text

###Configuring aa Caching Name Server
Caching nameserver

Caching nameservers store DNS query results in a local cache and removes resource records from the cache when their TTLs expire. This greatly imporves the efficiency of DNS name resolutions by reducing DNS traffic across the internet. 


DNSSEC validation

Prevents cache poisoning. DNSSEC validation enabled allows the authenticity and integrity of resource records to be validated prior to being placed in the cache for use by clients.

####Configuring and administering unbound as a caching nameserver
Install unbound
```bash
yum install -y unbound
```
Enable and start the service
```bash
systemctl start unbound.service
systemctl enable unbound.service
```
Configure the network interface to listen on

By default, unbound only listens on the localhost network interface. To make unbound available to remote clients as a caching nameserver, use the interface option in the server clause of /etc/unbound/unbound.conf to specify th enetwork interface(s) to listen on. 
```bash
interface: 0.0.0.0
```

Configure client access

By default, unbound refuses recursive queries from all clients. In the server clause of /etc/unbound/unbound.conf, use the access-control option to specify which clients are allowed to make recursive queries. 
```bash
access-control: 172.25.0.0/24 allow
```

Configure forwarding

For a caching nameserver, forward all queries by specifying a forward-zone of ".".
```bash
forward-zone:
	name: "."
	forward-addr: 172.25.254.254
```

If desired, bypass DNSSEC validation for select unsigned zones. The domain-insecure option in the server clause of the file can be used to specify a domain for which DNSSEC validation should be skipped.
```bash
domain-insecure: example.com
```

If desired, install trust anchors for select signed-zones without complete chain of trust. Obtain the DNSKEY record for the key signing key of the zone using dig and input it as the value for the trust-anchor option
```bash
dig +dnssec DNSKEY example.com
```
```bash
trust-anchor: "example.om 2500 in DNSKEY 243 3 8 ;laksjd;lfkja;sdklfja;slkdfja;lskdjf;alksjdf;laksdjf;lkajsd;lkfjas;dklfja;sdklfj;aiowej;fiajsf;awiejf;alj"
```

Verify the configuration file:
```bash
unbound-checkconf
```

Restart services, configure firewall to allow DNS.
```bash
systemctl restart unbound.service
firewall-cmd --permanent --add-service=dns
firewall-cmd --reload
```

Dumping and loading unbound cache
```bash
unbound-control dump_cache
```

Flusing unbound cache
```bash
unbound-control flush www.example.com
```

###DNS Troubleshooting
Useful commands:
```bash
getent hosts example.com
gethostip example.com
dig A example.com
```
####DNS response codes:

* SERVFAIL - Failure of the DNS server to communicate with the nameservers authoritative for the name being queried.

* NXDOMAIN - No records were found associated with the name queried.

* REFUSED - DNS server has a policy restriction which keeps it from fulfilling the client's query.


## Configuring Email Transmission (PostFix) - Complete
###Concepts
####Null Clients
In practice, most servers are monitored and send out mails when incidents occur. This is often requires a configured /usr/sbin/sendmail to send emails to notify the respnsible system admins by using the corporate SMTP server. 
A 'null client' is a client machine that runs a local mail server which forwards all emails to an outbound mail relay for delivery. A null client does not accept local delivery for any messages, it can only send them to the outbound mail relay. Users may run mail clients on the null client ot read and send emails. 
The following are true on a null client:

* the sendmail command and programs that use it forward all emails to an existing outbound mail realy for delivery

* The local Postfix service does not accept local delivery for any email messages

* Users may run mail clients on the null client to read and send mails.

###Important and Configuration Files with directives.

* /etc/postfix/main.cf

	* inet_interfaces= Controls which network interfaces Postfix listens on for incoming and outgoing messages. If set to 'loopback-only', Postfix listens only on 127.0.0.1 and ::1. If set to all, Postfix listens on all network interfaces. One or more host names and IP addresses, separated by white space, can be listed.

	* myorigin= Rewrite locally posted email to appear to come from this domain. This helps ensure responses return to the correct domain the mail server is responsible for. 

	* relayhost= Forward all message to the mail server specified that are supposed to be sent to foreign mail addresses. Square brackets around the host name suppress the MX record lookup.

	* mydestination= Configure which domains the mail server is an end point for. Email addressed to these domains are delivered into local mailboxes.

	* local_transport= Determine how email addressed to $mydestination should be delivered. By default, set to local:$myhostname, which uses the local mail delivery agent to deliver incoming mail to the local message store in /var/spool/mail.

	* mynetworks= Allow relay through this mail server from a comma-separated list of IP addresses and networks in CIDR notation to anywhere, without further authentication. 

*/var/log/maillog - Logs

###Commands
View and change postfix directives with the postconf command
```bash
postconf [directive .. ..]
```
```bash
postconf -e '\<directive\> = \<value\>'
```
To show directives that have been changed from the default
```bash
postconf -n
```


## Providing Remote Block Storage
###Introduction to iSCSI
####Terminology: 

* initiator: An iSCSI client, typically available as software but also implemented as iSCSI HBAs. Initiators must be given unique names (see IQN)

* target: An iSCSI storage resource, configured for connection from an iSCSI server. Targets must be given unique names (see IQN). A target provides one or more numbered block devices called logical units (see LUN). An iSCSI server can provide many targets concurrently

* ACL: An access controll list entry, an access restriction unsing the node IQN(commonly the iSCSI Initiator Name) to validate access permissions for an initiator.

* discovery: Querying a target server to list configured targets. Target use requires an additional access steps

* IQN: An iSCSI Qualified Name, a worldwide unique name used to identify both initiators and targets, in the mandated naming format:

```bash
iqn.YYYY-MM.com.reversed.domain[:optional_string]
```

* login: Authenticating to a target or LUN to begin client block device use.

* LUN: A Logical Unit Number, numbered block devices attached to and available through a target. One or more LUNs may be attached to a single target, although typically a target provides only one LUN. 

* node: any iSCSI initiator or iSCSI target, identified by it's IQN. 

* portal: An IP address and port on a target or initiator used to establish connections. Some iSCSI implementations use portal and node interchangeably 

* TPG: Target Portal Group, the set of interface IP addresses and TCP ports to which a specified iSCSI target will listen. Target configuration can be added to the TPG to coordinate settings for multiple LUNs

###iSCSI target configuration
####Target server configuration demo
```bash
yum -y install targetcli
```

* Create backing storage(backstores)

```bash
block/ create block1 /dev/vdb2
```

* Create an IQN for the target

```bash
create iqn.2014-06.com.example:remotedisk1
```

* In the TPG, create an ACL for the client node to be used later.

```bash
cd iqn.2014-06.com.example:remotedisk1/tpg1
acls/ create iqn.2014-06.com.example:desktop0
```

* In this TPG, create a LUN for each existing backstores

```bash
luns/ create /backstores/block/block1
```

* Still inside the TPG, create a portal configuration to designate the listening IP address and ports.

```bash
portals/ create 172.25.0.11
```

* Add a port exemption to the firewall for port 3260

```bash
firewall-cmd --permanent --add-port=3260/tcp
firewall-cmd --reload
```

* Enable the target.service systemd unit.

```bash
systemctl enable target
```

###Accessing iSCSI Storage
Restart the iscsi service on the initiator
```bash
yum install iscsi-initiator-utils
systemdctl restart iscsi
```

Perform discovery with the following command:
```bash
iscsiadm -m discovery -t sendtargets -p <target-server>[:port]
```

To use the target, log in using the following command:
```bash
iscsiadm -m node -T iqn.2014-06.com.example:serverX [-p target_server[:port]] -l
```

Log out:
```bash
iscsiadm -m node -T <iqn-target-name> [-p target_server[:port]] -u
```

Delete a node record permantely:
```bash
iscsiadm -m node -T <iqn-target-name> [-p target-server[:port]] -o delete
```


## Providing File-based Storage
###NFS Exports
The /etc/exports file lists the directory to share to client hosts over the network and indicates which hosts or networks have access to the export. The following are valid nfs exports

```bash
/myshare	server0.example.com
/myshare	*.example.com
/myshare	server[0-20].example.com
/myshare	172.25.0.0/16
/myshare	2000:472:18:b51:c32:a21
/myshare	2000:472:18:b51::/64
/myshare	desktop0.example.com(ro,no_root_squash)
```
Don't forget to allow NFS traffic (port 111 and 2049, TPC and UDP) and reload the exports
```bash
exportfs -r
firewall-cmd --permanent --add-service=nfs
firewall-cmd --reload
```

###Protecting NFS Exports
####Security Methods
NFS clients must connect to the exported share using one of the methods mandated for that share, specified as a mount option sec=method. 

* none: Anonymous access to the files, writes to the server will be allocated UID and GID of nfsnobody. This requires the SELinux Boolean nfsd_anon_write to be active. 

* sys: File access based on standard Linux file permissions for UID and GID values. If not specified, this is the default. The NFS server trusts any UID sent by the client.

* krb5: Clients must prove identity using Kerberos and then standard Linux file permissions apply. UID/GID is determined based upon the Kerberos principal from the accessing user.

* krb5i: Adds a cryptographically strong guarantee that the data in each request has not been tampered with UID/GID is determined based upon the Kerberos principal from the accessing user.

* krb5p: Adds encryption to all requests between the client and the server, preventing data exposure on the network. This will have a performance impact, but provides the most security. UID/GID is dtermined based upon the Kerberos principal from the accessing user. 

nfs-secure-server needs to be running in addition to nfs-server on the nfs server. On the client, nfs-secure needs to be running.
NOTE: Kerberos options will require, at a minimum a /etc/krb5.keytab. Both the client and the server need this.

Example export with kerberos
```bash
/securedexport	*.example.com(sec=krb5p,rw)
```

####SELinux and labeled NFS
By default, NFS mounts on the client side have the SELinux context nfs_t, independent of the SELinux context they have on the server that provides the export. 
This behavior can be changed on the client side by using the mount option 'context=selinux_context'

Switching to NFSv4.2 will cause the SELinux context of a share to be exported. In /etc/sysconfig/nfs:

```bash
RPCNFSDARGS="-V 4.2"
```
Then restart the server

On the client side, mount -o v4.2 must be specified as the mount option.
```bash
mount -o sec=krb5p,v4.2 serverX:/securedexport /mnt/securedexport
```

Or if you want to mount persistantly, in the /etc/fstab:
```bash
serverX:/securenfs /mnt/secureshare nfs defaults,v4.2,sec=krb5p 0 0
```

###Providing SMB File Shares
Install Samba
```bash
yum install samba
```

####SeLinux context and booleans
If the shared directory will only be accessed through Samba, then the directory and all it's subdirectories and files should be labeled samba_share_t. Also, configure it so that restorecon will set this type on the share and it's contents.
```bash
semange fcontext -a -t samba_share_t '/sharedpath(/.*)?'
restorecon -vvFR /sharedpath
```
####Configuring /etc/samba/smb.conf
Under the [global] section,

* workgroup: Used to specify the Windows workgroup for the server.

```bash
workgroup = WORKGROUP
```

* security: controls how clients are authenticated by Samba. With security = user clients log in with a valid username and password

* host allow: comma,space, or tab delimted list of hosts that are permited to access the Samba server. If it is not specified, all hosts can access Samba. 

```bash
host allow = 172.25.0.0/24
```

#####File share sections

* path: must be set to indicate which directory to share

* writable: if all users should have write access

* write list: A list of users with write access

* valid users: A list of users allowed to access the share. If it's blank, all users can access the share. 

For example:
```bash
[myshare]
	path = /sharedpath
	writable = no
	valid users = fred, @management
```
@ specifies a group

#####The [homes] section
The home section defines a special file share, which is enabled by default. It makes local home directories available via SMB.
```bash
[homes]
	comment = Home Directories
	read only = No
	browseable = No
```
The "samba_enable_home_dirs" SELinux boolean must be on for this to work.
```bash
setsebool -P samba_enable_home_dirs=on
```
#####Validating /etc/samba/smb.conf
```bash
testparam
```

####Preparing Samba users
The 'security = user' setting requires a Linux account with a Samba account that has a vlid NTLM passwords. To create a Samba-only system user, keep the Linux password locked, and set the login shell to /sbin/nologin.

For example, to create the locked Linux account for user fred:
```bash
useradd -s /sbin/nologin fred
yum install samba-client
smbpasswd -a fred
```

####Starting Samba
```bash
systemctl start smb nmb
systemctl enable smb nmb
firewall-cmd --permanent --add-service=samba
firewall-cmd --reload
```

####Mounting SMB file systems
```bash
mkdir /mnt/myshare
mount -o username=fred //serverX/myshare /mnt/myshare
```

####Performing a Multiuser SMB Mount
cifscred is used to stash users credentials into a kernel managed keyring. This is how users are able to access a mount mounted by root. Root must mount the share with the multiuser option.

For example, for root:
```bash
mkdir /mnt/multiuser
mount -o multiuser,sec=ntlmssp,username=fred //serverX/myshare /mnt/multiuser
yum install cifs-utils
```
For frank:
```bash
cifscreds add serverX
<prompt for password>
echo "Frank was here" > /mnt/multiuser/frank2.txt
```

## Configuring MariaDB Databases
Installation
```bash
yum groupinstall mariadb mariadb-client -y
mysql_secure_installation
```
###SQL Commands for getting around
```mysql
USE mysql;
SHOW TABLES;
DESCRIBE <table>
INSERT INTO <table-name> (column1, column2, ..) VALUES ('value1', value2, ..)
DELETE FROM <table-name> WHERE id = 1
SELECT column1,column2 FROM <table-name>
SELECT * FROM <table-name>
```

###SQL Commands fro Users and Access Rights
```mysql
CREATE USER mobius@localhost IDENTIFIED BY 'password';
GRANT SELECT,UPDATE,DELETE,INSERT on <table-name,table-name> to mobius@localhost;
GRANT ALL PRIVILEGES ON *.* to username@hostname
```
###Backups
####Logical Backups
Dump it
```bash
mysqldump -u root -p password <table-name> > /backup/inventory.dump
```
Restore it
```bash
mysql -u root -p <table-name> < /backup/inventory.dump
```

####Physical Backups(Using LVM snapshots)
Involves taking a snapshot of the LVM the data base information is on. Then, flush tables and lock them
```mysql
FLUSH TABLES READ LOCK;
```
On a seperate terminal, snapshot the logical volume:
```bash
lvcreate -L20G -s -n mariadb-backup <logical-volume>
```
Unlock tables
```mysql
UNLOCK TABLES;
```
The snapshot can not be mount at an arbitrary location

## Providing Apache HTTD Web Service
###Basic Apache HTTPD configuration
/etc/httpd/conf/httpd.conf

Important Blocks and Configs:
```bash

* <Directory [directory]>: sets configuration directives for the specified directory, and all decendent directories. 
	
	Common directives inside this block include:

	* AllowOverride None: .htaccess files will not be consulted for per-directory configuration settings. Setting this to any other setting will have a performance penalty. 

	* Require All Denied: httpd will refuse to servce content out of this directory

	* Require All Granted: Allow access to this directory

	* Options [[+|-]OPTIONS].. : Turn on (or off) certain options for a directory. For example, the Indexes option will show a directory listng if a directory is requested and no index.html file exists in that directory. 

* DocumentRoot <directory>: This setting dtermines where httpd will search for requested files. It is important that the directory specified here is both readable by httpd(both regular and SELinux)

* <Files [file]>: works just as a <Directory> block, but here options for individual files is used.

* ErrorLog <file>:

* IncludeOption [directory/*.conf] : Works the same as regular include, but if no files are found, no error is generated.

* CustomLog "log-path" combined - Define custom log location
```
Starting the service, enabling the firewall
```bash
systemctl enable httpd.service
systemctl start httpd.service
firewall-cmd --permanent --ad--service=http --add-service=https
firewall-cmd --reload
```

Using an alternate document root:

New document root must be readable by the apache user/group. The SELinux context may have to be changed. the /srv/*/www/ directories already have rules in place to relabel these files. If a new rules needs to be added:
```bash
semanage fcontext -a -t httpd_sys_content_t '/new/location(/.*)?'
```

Sometimes you want web devs to have write access to document root. To do this, use facls.


###Configuring and Troubleshooting Virtual Hosts
Virtual hosts allow a single httpd server to servce content for multiple domains. Based on either the IP address of the server that was connected to, the hostname request by the client in the httpd request, or a combination of both.

Virtual hosts are configured using <VirtualHost> blocks inside the main configuration. 
```bash
<VirtualHost 192.168.0.1:80>
	DocumentRoot /srv/site1/www

	ServerName site1.example.com

	ServerAdmin webmaster@site1.example.com

	ErrorLog "logs/site1_error_log"

	CustomLog "logs/site1_access_log" combined
</VirtualHost>
```
####Wildcards and Priority
When a request comes in, httpd will first try to match aginst virtual hosts that have an explicit IP address set. If those matches failt, virtual hosts with a wildcard IP address are inspected. If there is still no match, the "main" server configuration is used. 

If no exact match has been found for a ServerName or ServerAlias directive, and there are multiple virtual hosts defined for the IP/port combination the request came in on, the first virtual host that matches an IP/port is used, with first being seen as the order in which virtual hosts are defined in the config file. 

When multiple *.conf files are used, they will be included in alphanumeric sorting order.

###Coniguring HTTPS
Using genkey 
```bash
genkey <FQDN>
```
This will generate a bunch of files:

* /etc/pki/tls/private/<fqdn>.key - The private key. NEEDS TO HAVE PERMISSIONS OF 0600

* /etc/pki/tls/certs/<fqdn>.0.csr - the file generated if you requested a signing reqeust. 

* /etc/pki/tls/certs/<fqdn>.crt - The public certificate

Configure a host with SSL
```bash
<VirtualHost *:443>
	ServerName demo.example.com
	SSLEngine on
	SSLProtocol all -SSLv2 -SSLv3
	SSLCipherSuite HIGH:MEDIUM:!aNull:!MD5
	SSLHonorCipherOrder on
	SSLCertificateFile /etc/pki/tls/certs/demo.example.com.crt
	SSLCertificateKeyFile /etc/pki/tls/private/demo.example.com.key
	SSLCertificateChainFile /etc/pki/tls/certs/example-ca.crt
</VirtualHost>
```

* SSLEngine on - This is the directive that actually turns on TLS for this virtual host

* SSLProtocol all -SSLv2 -SSLv3 : This directive specifies the list of protocols that htppd is willing to speack with clients. 

* SSLCipherSUITE HIGH:MEDIUM:!aNull:!MD5 - This directive lists what encryption ciphers httpd is willing to use when communicating with clients. 

* SSLCertificateFile <file> - This directive instructs httpd where it can read the certificate for this virtual host

* SSLCertificateKeyFile <file> - This directive instructs httpd where it can read the private key for this virtual host.

* SSLCertificateChainFile <file> - a copy of all CS certificates used in the signing process concatentated together.

###Configuring HTTP Strict Transport Security
Automatically redirect clients connecting over http to the same resource using https
```bash
RewriteEngine on
RewriteRule ^(/.*)$ https://%{HTTP_POST}$1 [redirect=301]
```
###Dynamic content
To have httpd treat a location as CGI executables, the following syntax is used 
```bash
ScriptAlias /cgi-bin/ "/var/www/cgi-bin/"
```

Serving dynamic php content

1. Install the mod_cgi package

2. Ensure the SetHandler directive is in the main config

Serving dynamic python content

1. Install the mod_wsgi package

2. Add a WSGIScriptAlias line to a virtual host definition

```bash
WSGIScriptAlias /myapp/ /srv/myapp/www/myapp.py
```
NOTE:

When a network connection to another needs to be made from within the web application, and the target is not a well-known database port, the SELinux Boolean httpd_can_network_connect must be set to 1. 

## Writing Bash Scripts
So easy...
Use these commands:
```bash
#Turn uppercase letters to lowercase
tr 'A-Z' 'a-z'
#Cut first field of a colon seperated list
cut -d: -f1
#Get first letter of any string
echo "someInput" | cut -c 1
#Count the number of matches in a file
grep -c ^someregex$ /some/file
```


## Bash Conditionals and Control Structures
Special variables
```bash
#See arguments as one word
$*
#See arguments as seperate words
$@
#Number of argumnets
$#
#See the exit status of an executed command
$?
```
Conditonal Structures
```bash
#If statements
if <CONDITION>; then
	<STATEMENT>
		...
	<STATEMENT>
elif <CONDITION>; then
	<STATEMENT>
else
	<STATEMENT>
fi

#Case statements
case "$1" in
	start)
		start
		;;
	stop)
		rm -f $lockfile
		stop
		;;
	restart)
		restart
		;;
	reload)
		reload
		;;
	*)
		echo "Usage..."
		;;
esac
```

## Configuring the Shell Environment
### Concepts
#### Environment Variables 
What makes variables environment variables is that they have been exported in the shell. The key to making a variable become an environment variable is flaggin it for export using the export command.
#### bash start-up scripts
Profiles are for setting and exporting of environment variables, as well as running commands that should only be run upon login. RCs, such as /etc/bashrc, are for running commands, setting aliases, defining functions, and other settings that cannot be exported to sub-shells. 

Usually, profiles are only executed ina login shell, whereas RCs are executed every time a shell is created.
#### Using alias
alias is a way administrators or users can define their own command to the system or override the use of existing system commands.
#### Using functions
function_name() {
 body
}


