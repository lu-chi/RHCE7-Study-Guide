# RHCE7-Study-Guide
My Study Guide for RHCE7



## Controlling Services and Daemons

##Concepts
### Unit Types

* Service Units: have a .service extension and represent system services. This type of unit is used to start frequently accessed daemons, such as a web server.

* Socket Units: have a .socket extension and represent interprocess communication (IPC) sockets. Control of the socket will be passed to a daemon or newly started service when a client connection is made. Socket units are used to delay the start of a service at boot time and to start less frequently used servies on demand. These are similarin principle to services which use the xinetd superserver to start on demand.

* Path Units: have a .path extension and are used to delay the activation of a service until a specific file system change occurs. This is commonly used for services which use spool directories, such as a printing system.

### Targets
A target is a set of systemd units that should be started to reach a desired state. 
#### Important targets
 
 * graphcial

## Commands
### Common systemctl commands
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



