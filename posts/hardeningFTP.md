<p style="text-align: center; font-size: 40px;">Hardening File Transfer Protocol Services</p>

---

##### Metadata

###### Last revised - 10/25/21

###### Author       - Mason Sipe

---
## Preamble

This post is about the different ways you can harden the vast FTP services available on Linux systems. These are all up for interpretation. However, they are to be used as a guide since there are many possibilities for each service. Tailor these settings to your specific needs, Do not copy and paste.

## Background:

The File Transfer Protocol has multiple uses throughout a specific network architecture. It is commonly used to store bulk data, backups, development projects, etc. The official FTP ports are command port 21 and data port 20. FTP uses two ports for connection to the device and the actual data transfer on another port. FTP is a client-server modeled service that uses authentication in cleartext (unencrypted), and depending on the service, can allow for anonymous access.

## Table of Contents

- [VSFTPD](#vsftpd)
    - [Disable Public Access](#disable-public-access)
    - [Local User Login](#local-user-login)
    - [Restricting Access to a User's Home Directory](#restricting-access-to-a-users-home-directory)
    - [Enabling SSL/TLS Encryption](#enabling-ssltls-encryption)
    - [Enable Logging](#enable-logging)
    - [Disabling Writing](#disabling-writing)
    - [Banner Settings](#banner-settings)
    - [Default UMASK](#default-umask)
    - [Anonymous Upload](#anonymous-upload)
    - [Anonymous Create Directories](#anonymous-create-directories)
    - [Directory Messages](#directory-messages)
    - [Port Transfers Originate Properly](#port-transfers-originate-properly)
    - [Specific Anonymous Ownership](#specific-anonymous-ownership)
    - [Idle Session Timeout](#idle-session-timeout)
    - [Data Session Timeout](#data-session-timeout)
    - [Unprivileged User Daemon](#unprivileged-user-daemon)
    - [Disable ABOR Requests](#disable-abor-requests)
    - [Ascii Mangling](#ascii-mangling)
    - [Customize Login Banner](#customize-login-banner)
    - [Deny Specific Email Addresses](#deny-specific-email-addresses)
    - [Listening](#listening)
    - [Other settings](#other-settings)
- [ProFTPD](#proftpd)
  - [Set Server Type](#set-server-type)
  - [Set a Server Name](#set-a-server-name)
  - [Set a UMASK](#set-a-umask)
  - [Set FTP Port](#set-ftp-port)
  - [Set Dedicated Unprivileged User](#set-dedicated-unprivileged-user)
  - [Default Display Message](#default-display-message)
  - [Defer New Connections to Delay Connections](#defer-new-connections-to-delay-connections)
  - [Use IPv6](#use-ipv6)
  - [Disable Ident Protocol](#disable-ident-protocol)
  - [Set a Maximum Amount of Child Processes](#set-a-maximum-amount-of-child-processes)
  - [Set a Maximum Amount of Connections](#set-a-maximum-amount-of-connections)
  - [Set a Maximum Toward Login Attempts](#set-a-maximum-toward-login-attempts)
  - [Set the Default Root](#set-the-default-root)
  - [Allowed Regular Expression](#allowed-regular-expression)
  - [Limit Per-Machine Connections](#limit-per-machine-connections)
  - [Limit User-Per-Machine Connections](#limit-user-per-machine-connections)
  - [Limit Host Per User](#limit-host-per-user)
  - [Resource Efficiency](#resource-efficiency)
    - [Limit CPU Usage For Session](#limit-cpu-usage-for-session)
    - [Limit Memory Usage For Session](#limit-memory-usage-for-session)
    - [Limit Memory Usage For Daemon](#limit-memory-usage-for-daemon)
  - [Hardening /proc Directory](#hardening-proc-directory)
  - [Disable Reverse DNS Lookups](#disable-reverse-dns-lookups)
  - [Strict File Listing](#strict-file-listing)
  - [Deny Overriding Server Settings](#deny-overriding-server-settings)
  - [Disable .ftpaccess  File Checking in Every Directory](#disable-ftpaccess-file-checking-in-every-directory)
- [PureFTPD](#pureftpd)
- [Sources](#sources)
### VSFTPD
(Very Secure FTP Daemon)

Configuraion = `/etc/vsftpd.conf` or `/etc/vsftpd/vsftpd.conf`


#### Disable Public Access

If you do not want anonymous users to log in, disable them.

`anonymous_enable=NO`

xmodulo claims: "Mandatory authentication is thus enabled, and only existing Linux users will be able to connect using their login credentials."

#### Local User Login

You can control if you want users to log in locally as well. You can choose to have it permitted or denied depending on your specific setup and situation.

`local_enable=YES/NO`


#### Restricting Access to a User's Home Directory

By default, users can access a remote directory as long as it is readable. This is not ideal in most situations as the users can download any readable files. You can disable that by setting this configuration option.

`chroot_local_user=YES`

#### Enabling SSL/TLS Encryption

This is to ensure a secure tunnel is formed upon connecting. FTP by default is in cleartext which is not a safe form of sensitive file transferring.

Assuming you have OpenSSL installed:

Debian:
`sudo openssl req -x509 -days 365 -newkey rsa:2048 -nodes -keyout /etc/vsftpd.pem -out /etc/vsftpd.pem`

CentOS/RHEL/Fedora:
`sudo openssl req -x509 -days 365 -newkey rsa:2048 -nodes -keyout /etc/vsftpd/vsftpd.pem -out /etc/vsftpd/vsftpd.pem`

According to xmodulo, this is what needs to be added/modified within the VSFTPD configuration file: 

```bash
# enable TLS/SSL
ssl_enable=YES

# force client to use TLS when logging in
allow_anon_ssl=NO
force_local_data_ssl=YES
force_local_logins_ssl=YES
ssl_tlsv1=YES
ssl_sslv2=NO
ssl_sslv3=NO
require_ssl_reuse=NO
ssl_ciphers=HIGH

# specify SSL certificate/private key (Debian/Ubuntu)
# For CentOS/Fedora/RHEL, replace it with /etc/vsftpd/vsftpd.pem
rsa_cert_file=/etc/vsftpd.pem
rsa_private_key_file=/etc/vsftpd.pem

# define port range for passive mode connections
pasv_max_port=65535
pasv_min_port=64000
```
#### Enable Logging

Accounting is mandatory within an organization for multiple reasons other than security; there are legal requirements and other forms of various reasons you would need this.

To enable logging to add/modify these settings:

```bash
xferlog_enable=YES
xferlog_std_format=NO
xferlog_file=/var/log/vsftpd.log 
log_ftp_protocol=YES
debug_ssl=YES
```

#### Disabling Writing

You can allow/deny writing to a directory with VSFTPD as well. To do this, you can modify:

`write_enable=YES/NO`

#### Banner Settings

If you are following the V-38599 policy with DoD, your server should show your banner.

`banner_file=/etc/issue`

#### Default UMASK

Changing the default UMASK permissions will help with default security.

`local_umask=022`

#### Anonymous Upload

You can allow for the anonymous user to write files if you so, please.

`anon_upload_enable=YES/NO`

#### Anonymous Create Directories

Along with writing files, they can also write directories if you choose.

`#anon_mkdir_write_enable=YES/NO`

#### Directory Messages

Directory messages when remote users enter specific directories is also a feature that is nice to have enabled.

`dirmessage_enable=YES`


#### Port Transfers Originate Properly

VSFTPD can negotiate ports as long as they are started from port 20.

`connect_from_port_20=YES`

####  Specific Anonymous Ownership

You can specify a specific owner for files that the anonymous user creates (be careful.)

`chown_uploads=YES`

`chown_username=whoever`

#### Idle Session Timeout

User inactivity can be dangerous if the computer is left unattended.

`idle_session_timeout=600`

#### Data Session Timeout

User inactivity from the data port to disconnect after no connection is not required.

`data_connection_timeout=120`

#### Unprivileged User Daemon

You can set an unprivileged, sandboxed, or otherwise untrusted user to be in control of the VSFTPD process in case of a service compromise.

`nopriv_user=ftpsecure`

#### Disable ABOR Requests

Older versions of VSFTPD use this setting. Disable it if not required.

`async_abor_enable=NO`

#### Ascii Mangling

According to user `fcaviggia` on Github:

```bash
# By default, the server will pretend to allow ASCII mode but in fact ignore
# the request. Turn on the below options to have the server actually do ASCII
# mangling on files when in ASCII mode.
# Beware that on some FTP servers, ASCII support allows a denial of service
# attack (DoS) via the command "SIZE /big/file" in ASCII mode. vsftpd
# predicted this attack and has always been safe, reporting the size of the
# raw file.
# ASCII mangling is a horrible feature of the protocol.
ascii_upload_enable=NO
ascii_download_enable=NO
```

#### Customize Login Banner

`ftpd_banner=<message>`

#### Deny Specific Email Addresses

This is an ambiguous command and is informal. Avoid blacklisting; however, if you specifically want to deny an address, you can do so with this:

`deny_email_enable=YES`

`banned_email_file=/path/to/blacklist`


#### Listening

You can specify if you want VSFTPD to run in standalone mode and listen on an IP Socket.

`listen=YES`

`listen_ipv6=YES`

If you are not using IPv6, disable this feature.

#### Other settings

```
pam_service_name=vsftpd
userlist_enable=YES
tcp_wrappers=YES
```
### ProFTPD 
Configuration = `/etc/proftpd//proftpd.conf`

#### Set Server Type

You can set the type of server between the different types available. A dedicated server should be a standalone server.

`ServerType standalone`

#### Set a Server Name

`ServerName "Corporation FTP Server"`

#### Set a UMASK

`Umask 022`

#### Set FTP Port

It is wise to change the default port of a specific service to avoid automated attacks.

`Port 21`

#### Set Dedicated Unprivileged User
```bash
User  nobody
Group nogroup
```

#### Default Display Message

`ServerIdent on "FTP server"`

#### Defer New Connections to Delay Connections

This is used to make the client prove their authenticity before connecting.

`DeferWelcome on`

#### Use IPv6

Determine if you need to use IPv6 Hosting

`UseIPv6 off`

#### Disable Ident Protocol

Ident is considered to be a dead or dying protocol. It is commonly used within IRC chats. [RFC1413](http://www.faqs.org/rfcs/rfc1413.html) for more details. 

`IdentLookups off`

#### Set a Maximum Amount of Child Processes


`MaxInstances 30`

#### Set a Maximum Amount of Connections

`MaxClients 10`

#### Set a Maximum Toward Login Attempts

`MaxLoginAttempts 3`

#### Set the Default Root

`DefaultRoot ~`

#### Allowed Regular Expression

`AllowFilter "^[a-zA-Z0-9 ,]*$"`

#### Limit Per-Machine Connections

`MaxClientsPerHost 1 "Sorry, you may not connect more than one time."`


#### Limit User-Per-Machine Connections

`MaxClientsPerUser 1 "Only one such user at a time.`

#### Limit Host Per User

`MaxHostsPerUser 1 "Sorry, you may not connect more than one time."`

#### Resource Efficiency

##### Limit CPU Usage For Session

` RLimitCPU session 10  `

##### Limit Memory Usage For Session
`  RLimitMemory session 4096`

##### Limit Memory Usage For Daemon

`  RLimitMemory daemon 8192 max`


#### Hardening /proc Directory

```bash
  <Directory /proc>
    <Limit ALL>
      DenyAll
    </Limit>
  </Directory>
```

#### Disable Reverse DNS Lookups

This is only useful for logging, which can be done when needed. Not while serving. 

`  UseReverseDNS off`


#### Strict File Listing

`  ListOptions +R strict`

#### Deny Overriding Server Settings

```bash
  ListOptions "" maxdepth 3
  ListOptions "" maxdirs 10
  ListOptions "" maxfiles 1000
```

#### Disable .ftpaccess  File Checking in Every Directory

`  AllowOverride off`

### PureFTPD

Configuration = `/etc/pure-ftpd/pure-ftpd.conf`

#### Chroot Everyone

This will make everyone see their files in their home directory. They cannot see other people's files or directories.

`ChrootEveryone yes`

#### No Anonymous Access

This denies any anonymous attempts to log in to the server without written credentials.

`NoAnonymous yes`

#### Max Clients

Set a limit to the number of clients that can connect simultaneously.

`MaxClientsNumber 50`

#### Max Clients Per IP

Set maximum connections for a single internet protocol address

`MaxClientsPerIP 1`

#### Idle Timeout

Computers left idle are dangerous to the remote connection for a physical session hijack.

`MaxIdleTime 1`

#### Limit Recursion

Set a limit to the files that will be listed when the `ls` command is executed.

`LimitRecursion 500 8`

#### Set a UMASK

Default file permissions are not secured. Change it to a guaranteed value.

`Umask 022`


---

###### Sources:

- ###### <https://www.xmodulo.com/secure-ftp-service-vsftpd-linux.html>
- ###### <https://github.com/fcaviggia/hardening-script-el6/blob/master/config/vsftpd.conf>
- ###### <https://linux.die.net/man/8/vsftpd>
- ###### <https://www.sysadmin.md/secure-existing-proftpd-server-installation.html>
- ###### <https://www.sysadmin.md/secure-existing-pureftpd-installation.html>
- ###### <https://likegeeks.com/ftp-server-linux/>
- ###### <https://security-24-7.com/hardening-guide-for-vsftpd-on-rhel-5-4/>
- ###### <https://www.ncftp.com/ncftp/>
- ###### <http://www.proftpd.org/docs/configs/basic.conf>
- ###### <http://www.proftpd.org/docs/howto/BCP.html>
---

###### [Home](https://mksipe.github.io/mksipe/)