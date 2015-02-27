# RHCE7-Study-Guide
My Study Guide for RHCE7



## Controlling Services and Daemons

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

1. Reboot the System, enter edit mode for the proper entry in the bootloader. 

2. Append rd.break to the end of the line that starts with linux16. Restart with Ctrl+X

3. Remount /sysroot as read-write. 

```bash
switch_root:/# mount -oremount,rw /sysroot
```

4. Switch into a chroot jail, where /sysroot is treated as the root of the file system tree.

```bash
chroot /sysroot
```

5. Set a new root password

6. Make sure that all unlabeled files (including /etc/shadow at this point) get relabeled during boot. 

```bash
touch /.autorelabel
```

7. Type exit twice to continue booting.
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


## Configuring Link Aggregation and Bridging


## Network Port-Security


## Managing DNS for Servers


## Configuring Email Transmission


##Providing Remote Block Storage


## Providing File-based Storage


## Configuring MariaDB Databases


## Providing Apache HTTD Web Service


## Writing Bash Scripts



## Bash Conditionals and Control Structures


## Configuring the Shell Environment



