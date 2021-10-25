<p style="text-align: center; font-size: 40px;">Hardening File Transfer Protocol Services</p>

---

##### Metadata

###### Last revised - 10/25/21

###### Author       - Mason Sipe

---
## Preamble

This post is about the different ways you can harden the vast FTP services that are available on linux systems. These are all up for interpretation, however, they are to be used as a guide since there are lots of possibilities for each individual service. Tailor these settings to your specific needs, Do not copy and paste.


## Table of Contents



## Background:

The File Transfer Protocol has multiple uses throughout a specific network architechture. It is commonly used to store bulk data, backups, development projects, etc. The official FTP ports are command port 21 and data port 20. FTP uses two ports for connection to the device, and the actual data transfer on another port. FTP is a client-server modeled service that uses authentication in clear-text (unencrypted), and depending on the service, can allow for anonymous access.

---

###### Sources:

- ###### <https://www.xmodulo.com/secure-ftp-service-vsftpd-linux.html>
- ###### <https://www.sysadmin.md/secure-existing-proftpd-server-installation.html>
- ###### <https://www.sysadmin.md/secure-existing-pureftpd-installation.html>
- ###### <https://likegeeks.com/ftp-server-linux/>
- ###### <https://security-24-7.com/hardening-guide-for-vsftpd-on-rhel-5-4/>


---

###### [Home](https://mksipe.github.io/mksipe/)