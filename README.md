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


## Providing File-based Storage



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


