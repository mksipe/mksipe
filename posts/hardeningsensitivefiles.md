<p style="text-align: center; font-size: 40px;">Hardening Sensitive Files</p>

---

##### Metadata

###### Last revised - 11/15/21

###### Author       - Mason Sipe

---

## Preamble

This post is designed to set awareness of the individual changes for files that you should protect on your system to keep and maintain a hardened system.

## Background:

Some files on Linux are by default readable by unprivileged users, which is a problem. Throughout the evolution of Linux, logs, core dumps, and other information have been stored on your system, which contains sensitive information that the system administrator should protect.

## Table of Contents

- [Common Executables](#common-executables)
- [List all setuid/setgid File on the System](#list-all-setuid/setgid-file-on-the-system)
- [List All Files Without a User or Proprietary Group](#list-all-files-without-a-user-or-proprietary-group)
- [Dedicated Temporary Directories](#dedicated-temporary-directories)
- [Sticky Bit File Attributes](#sticky-bit-file-attributes)
### Common Executables

According to ANSSI, these files must be protected and verified to maintain proper file permissions after each update.

|Executable|Comment|
|:-|-:|
|`/bin/mount`|To disable, unless absolutely necessary for users.|
|`/bin/netreport1`|To disable.|
|`/bin/ping6/`|(IPv6) Same as ping.|
|`/bin/ping`|(IPv4) Remove setuid right, unless a program requires it for monitoring.|
|`/bin/su`|Change user. Do not disable.
|`/usr/bin/sudoedit`|Same as sudo.|
|`/usr/bin/wall`|To disable.|
|`/usr/bin/X`|To disable unless the X server is required.|
|`/usr/lib/dbus-1.0/dbusdaemon-launch-helper`|To disable when D-BUS is not used.|
|`/usr/lib/openssh/sshkeysign`|To Disable.|
|`/usr/lib/pt_chown`|To disable (allows to change the owner of PTY before the existence of devfs).|
|`/usr/libexec/utempter/utempter`|To disable if the utempter SELinux profile is not used.|
|`/usr/sbin/exim4`|Disable if Exim is not used.|
|`/usr/sbin/suexec`|Disable if suexec Apache is not used.|
|`/usr/sbin/traceroute`|(IPv4) Same as ping.|
|`/usr/sbin/traceroute6`|(IPv6) Same as ping.|


### List all setuid/setgid File on the system

`find / -type f -perm /6000 -ls 2>/dev/null`

You can remove the setuid/setuid propertied from these files with

`chmod u-s <fichier >`  Remove the setuid bit

`chmod g-s <fichier >`  Remove the setgid bit


### List All Files Without a User or Proprietary Group


`find / -type f \( -nouser -o -nogroup \) -ls 2>/dev/null`

### Dedicated Temporary Directories

By default, the /tmp directory is readable and writable to everyone on the system, regardless of user or group.

It is recommended to install one of these packages to mitigate this: `pam_mktemp`, `pam_namespace`, `pam_tmpdir`.

### Sticky Bit File Attributes

The sticky bit, according to technopedia, is "a permission bit which is set on a file or folder, thereby permitting only the owner or root user of the File or folder to modify, rename or delete the concerned directory or File. No other user would be allowed to have these privileges on a file that has a sticky bit.
"
This command shows files that anybody can modify without a sticky bit.

`find / -type d \( -perm -0002 -a \! -perm -1000 \) -ls 2>/dev/null`

same command, but with root ownership

`find / -type d -perm -0002 -a \! -uid 0 -ls 2>/dev/null`

find files that anyone can modify

`find / -type f -perm -0002 -ls 2>/dev/null`

You can enable the sticky bit with the chmod command.

chmod +t <file/dir(RECURSIVE)> 



###### Sources:

- ###### <https://www.ssi.gouv.fr/en/guide/configuration-recommendations-of-a-gnulinux-system/>
- ###### <https://www.techopedia.com/definition/22339/sticky-bit>

---

###### [Home](https://mksipe.github.io/mksipe/) | [Privacy Policy](https://mksipe.github.io/mksipe/Privacy)