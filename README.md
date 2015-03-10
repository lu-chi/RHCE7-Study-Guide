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



## Configuring Link Aggregation and Bridging


## Network Port-Security


## Managing DNS for Servers


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

	*relayhost= Forward all message to the mail server specified that are supposed to be sent to foreign mail addresses. Square brackets around the host name suppress the MX record lookup.

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

```shell
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
```shell
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
```shell
/securedexport	*.example.com(sec=krb5p,rw)
```

####SELinux and labeled NFS
By default, NFS mounts on the client side have the SELinux context nfs_t, independent of the SELinux context they have on the server that provides the export. 
This behavior can be changed on the client side by using the mount option 'context=selinux_context'

Switching to NFSv4.2 will cause the SELinux context of a share to be exported. In /etc/sysconfig/nfs:
```shell
RPCNFSDARGS="-V 4.2"
```
Then restart the server

On the client side, mount -o v4.2 must be specified as the mount option.
```bash
mount -o sec=krb5p,v4.2 serverX:/securedexport /mnt/securedexport
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

```shell
workgroup = WORKGROUP
```

* security: controls how clients are authenticated by Samba. With security = user clients log in with a valid username and password

* host allow: comma,space, or tab delimted list of hosts that are permited to access the Samba server. If it is not specified, all hosts can access Samba. 

```shell
host allow = 172.25.0.0/24
```

#####File share sections

* path: must be set to indicate which directory to share

* writable: if all users should have write access

* write list: A list of users with write access

* valid users: A list of users allowed to access the share. If it's blank, all users can access the share. 

For example:
```shell
[myshare]
	path = /sharedpath
	writable = no
	valid users = fred, @management
```
@ specifies a group

#####The [homes] section
The home section defines a special file share, which is enabled by default. It makes local home directories available via SMB.
```shell
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


## Providing Apache HTTD Web Service


## Writing Bash Scripts



## Bash Conditionals and Control Structures


## Configuring the Shell Environment
### Concepts
#### Environment Variables 
What makes variables environment variables is that they have been exported in the shell. The key to making a variable become an environment variable is flaggin it for export using the export command.
#### bash start-up scripts
Usually, profiles are only executed ina login shell, whereas RCs are executed every time a shell is created.
#### Using alias
alias is a way administrators or users can define their own command to the system or override the use of existing system commands.
#### Using functions
function_name() {
 body
}


