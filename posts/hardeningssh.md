<p style="text-align: center; font-size: 40px;">Hardening SSH</p>

---

##### Metadata

###### Last revised - 10/2/21

###### Author       - Mason Sipe

---
## Preamble

This is a post about the various options you have in regards to your SSH configuration. Please note that the service you are running depends entirely on your usage and must be manually inspected and applied to your need. DO NOT COPY AND PASE SETTINGS. Read before you commit.


## Table of Contents

- [Hardening SSH Service](#hardening-ssh-service)
    - [X11 Forwarding](#usage-of-x11-forwarding)
    - [rhosts](#disable-rhosts)
    - [DNS](#dns-hostname-checking)
    - [Empty Passwords](#disable-empty-passwords)
    - [AuthenticatIOn Attempts](#maximum-authentication-attempts)
    - [Public Key Authentication](#public-key-Authentication)
    - [Root Login](#disable-root-login)
    - [Protocol](#set-the-ssh-protocol)
    - [User and Group restriction](#allowusers-denyusers-allowgroups-and-denygroups)
    - [Hashing Known Hosts Locally](#use-hashknownhosts)
    - [Port](#port)
    - [Change Idle LoginTimeout](#change-default-grace-time)
    - [Maximum Users Connections](#clientalivecountmax)
    - [Client Idle Timeout](#clientaliveinterval)
    - [Sandboxing Processes](#sandbox-privileges)
    - [Enable Logging](#authentication-logging)
    - [Local File Authentication](#strict-modes)
    - [PAM](#using-pam)


### Server Configuration Options

> #### Configuration files are case sensitive. If there is an incorrect entry, the ssh service will stop. You can test the SSH configuration with `sshd -T`

### These are the possible options (at the time of this writing) ssh used according to <https://www.ssh.com/academy/ssh/config>.


### Hardening SSH Service

> Important note
: Check the SSH service frequently and make changes slowly and before deployment. You can do this by running `systemctl status ssh`


> Check for active conncections : `ss -n -o state established '(dport = :22 or sport = :22'` 

> SSH configuration file : `/etc/ssh/sshd_config`

#### Usage of X11 Forwarding
: If X11 is not needed, Disable it.

`X11Forwarding no`

#### Disable rhosts
: Rhosts was a weak form of authentication when SSH was first widely used. It made it so that it would be trusted to connect if a machine were on the same IP range.

`IgnoreRhosts yes`


#### DNS Hostname Checking
: DNS hostname checking will be a safeguard if you have an internal DNS server set up. Be careful if there is no internal DNS; this can add timeouts and other networking issues. 

`UseDNS yes`

#### Disable Empty Passwords
: Accounts should not be able to be accessed remotely without accountability. Restrict accounts password policy by prohibiting empty passwords.

`PermitEmptyPasswords no`

#### Maximum Authentication Attempts
: Slow down and prevent Brute force attacks on user enumeration and password attempts. 

`MaxAuthTries 3`

#### Public Key Authentication
: If you choose not to use a password or want additional authentication for remote connections, you can allow public-key authentication.

`PubkeyAuthentication yes`

#### Disable Root Login
: It is best not to allow root to be allowed to login remotely. A user should be established to enable authentication and accounting, then they, individually, can use Sudo.

`PermitRootLogin no`

#### Set the SSH Protocol
: Use the latest version of SSH.

`Protocol 2`

#### AllowUsers, DenyUsers, AllowGroups and DenyGroups
: You can deny ssh access to specific users and allow particular groups. The easiest way to manage this is with AllowGroups and add users to that and deny all else. However, as stated before, do as your specific situation needs. The concept of an implicit denial should be applied here to deny anyone who is not on a whitelist.

> Side note, according to [ServerFault](https://serverfault.com/questions/617081/how-to-use-both-allowgroups-and-allowusers-in-sshd-config) the order of these entries are processed in the following: DenyUsers, AllowUsers, DenyGroups, 

`AllowUsers user1 user2`

`AllowGroups ssh-users`

#### Use HashKnownHosts
: When a successful connection is made with SSH, the connection information is stored in the user directory `.ssh`. However, this can be a risk in case of a compromise. You can hash this information using the HashKnownHosts option.

`HashKnownHosts ssh`

#### Port
: You can manually change the port SSH uses upon startup. Changing the default port is an excellent way to prevent bots and users from directly targetting your ssh port based upon default configuration knowledge.

`port 42451`

#### Change Default Grace Time
: By default, you have two minutes to log in. After those two minutes, the server will terminate the connection.

`LoginGracetime 60`


#### ClientAliveCountMax
: This setting checks the real, alive connection and sets a limit to it.

`ClientAliveCountMax 5`


#### ClientAliveInterval
: This setting indicates the timeout in seconds. After a set of seconds, the server will send a message to the client asking for a response. 

`ClientAliveInterval 600`

#### Sandbox Privileges
: You can sandbox users' authentication to create trusted and untrusted processes on the server. i.e., a user is thrown into a sandboxed environment upon logging in; then, once credentials are verified, the daemon will put them into a legitimate process.

`UsePrivilegeSeparation sandbox`


#### Authentication Logging
: You can specify if you want logging enabled upon the daemon, authentication, or authentication privilege.

`SyslogFacility AUTHPRIV`
: You can also specify the logging intensity.

`LogLevel VERBOSE` - shows all logs

> The possible values are: QUIET, FATAL, ERROR, INFO, VERBOSE, DEBUG, DEBUG1, DEBUG2 and DEBUG3


#### Strict Modes
: This option specifies whether ssh should check the file modes and ownership of the user's files and home directory before accepting login.

`StrictModes yes`

#### Using PAM
: You can choose to use Pluggable Authentication Modules if you prefer. However, some other settings automatically enable when you do this, leading to security complications, such as ChallengeResponseAuthentication being changed to yes. As stated, if not required, do not use.

`UsePAM yes`

---

###### Sources:
###### - https://serverfault.com/questions/617081/how-to-use-both-allowgroups-and-allowusers-in-sshd-config
###### - https://www.ssh.com/academy/ssh/config
###### - https://linux-audit.com/audit-and-harden-your-ssh-configuration/
###### - https://www.thegeekstuff.com/2011/05/openssh-options/

---

###### [Home](https://mksipe.github.io/mksipe/)