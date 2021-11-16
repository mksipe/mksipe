<p style="text-align: center; font-size: 40px;">Hardening SSH</p>

---

##### Metadata

###### Last revised - 11/15/21

###### Author       - Mason Sipe

---
## Preamble

This post is to show the different options toward hardening the default password policies on a Linux system. Tailor these examples to your specific needs.


## Table of Contents

- [Login Defaults](#login-defaults)
    - [Max Password Age](#set-the-maximum-password-expire)
    - [Min Password Age](#set-the-minimum-password-expire)
    - [Min Password Length](#set-the-minimum-password-length)
    - [Password Change Warning](#set-a-password-expire-warning)
    - [Using a Safer UMASK](#using-a-safer-umask)
- [Password Complexity](#setting-password-complexity-with-pam)
    - [Common Password](#common-password-file)
    - [User Account Control](#set-user-access-control)
    - [Set User Limits](#set-user-limits)
    - [Temporary Files and Permissions](#temporary-directories-and-pam)
    - [Undefined PAM Applications](#configure-undefined-pam-applications)
    - [ Automatic Accounts Blocking](#automatic-accounts-blocking)
    - [Blocking Root Access](#blocking-root-access)
- [Defining](#applying-new-password-settings-to-a-user)
    - [Viewing User Password Information](#viewing-password-information)
    - [Change Expire Date](#change-expire-date)
    - [Locking an Account](#locking-an-account)
    - [All Values](#all-values)
- [Sources](#sources)
    
### Login Defaults

The `/etc/login.defs` file is designed to apply default values to a user when they are not previously defined. ***These values are not used unless a user is created.***

#### Set the Maximum Password Expire

Setting the maximum time for a password to be used enhances security by forcing users to change their password after logging in after x days.

`PASS_MAX_DAYS 60`

#### Set the Minimum Password Expire

Setting the minimum amount of time until a password can help in case of a compromise and a user's password can still be used; at this point, an administrator would have to change it. 

`PASS_MIN_DAYS 7`

#### Set the Minimum Password Length

Setting a specific length can help make password cracking take longer by brute force, thus, making it more secure.

`PASS_MIN_LEN 12`

#### Set a Password Expire Warning

Warn users upon an upcoming password change.

`PASS_WARN_AGE 5`

#### Using a Safer UMASK

Changing the umask chooses the default file permissions, which is 077

The UMASK should be changed to retain privacy and security.

`UMASK 027` - `UMASK 022` is also preferred.

The UMASK should also be changed in the /etc/rc.d/rc file as well


#### Setting Encryption Method

You can manually set the encryption method in login.defs.

`ENCRYPT_METHOD SHA512`

You can also set the minimum rounds of that specific method.

`SHA_CRYPT_MIN_ROUNDS 65536`

### Setting Password Complexity with PAM

You can also specify specific requirements with PAM to only allow passwords that have particular characters in them.

You need to have the `libpam-cracklib` package installed for this section to work.  

#### Common Password File

The common password file sets the specific requirements to allow specific passwords to be used.

`/etc/pam.d/common-password`


```
password    requisite     pam_cracklib.so try_first_pass retry=3
password    sufficient    pam_unix.so md5 shadow nullok try_first_pass use_authtok
password    required      pam_deny.so obscure sha512 rounds=65536
```

> It is suggested to backup your pam files. If you do not, you may not be able to authenticate.


You need to add the following after `requisite pam_cracklib.so`

`retry=3 minlength=16 lcredit=-1 ucredit=-1 dcredit=-1 ocredit=-1 difok=4`

The file should look like this:

```
password    requisite     pam_cracklib.so try_first_pass retry=3 minlength=16 lcredit=-1 ucredit=-1 dcredit=-1 ocredit=-1 difok=4 ocredit=0 mexrepeat=-1 maxsequence=1 gecoscheck reject_username enforce_for_root
password    sufficient    pam_unix.so md5 shadow nullok try_first_pass use_authtok sha512
password    required      pam_deny.so
```

This addition makes it so that passwords now are required to be at least 16 characters long, need a lowercase, uppercase, digit, unique character, and no dictionary words within a password. 

#### Set User Access Control

The root user can only log into the system locally from terminals.

`/etc/pam.d/login`

`  auth     requisite  pam_securetty.so`


#### Set User Limits

"If you want to protect su, so that only some people can use it to become root on your system, you need to add a new group "wheel" to your system (that is the cleanest way, since no file has such a group permission yet). Add root and the other users that should be able to su to the root user to this group. Then add the following line to /etc/pam.d/su: " (Prev Securing Debian Manual).

`/etc/pam.d/login`

`session  required   pam_limits.so`

`  auth        requisite   pam_wheel.so group=wheel debug`


`/etc/pam.d/ssh`

`  auth        required    pam_listfile.so item=user sense=allow file=/etc/sshusers-allowed onerr=fail`

#### Temporary Directories and PAM

This module requires the `libpam-tmpdir` package. This package will prevent insecure file exposure, i.e., https://www.debian.org/security/2005/dsa-883.

`/etc/pam.d/common-session`

` session    optional     pam_tmpdir.so`


##### Configure Undefined PAM Applications

`/etc/pam.d/other`

```
  auth     required       pam_securetty.so
  auth     required       pam_unix_auth.so
  auth     required       pam_warn.so
  auth     required       pam_deny.so
  account  required       pam_unix_acct.so
  account  required       pam_warn.so
  account  required       pam_deny.so
  password required       pam_unix_passwd.so
  password required       pam_warn.so
  password required       pam_deny.so
  session  required       pam_unix_session.so
  session  required       pam_warn.so
  session  required       pam_deny.so
```

##### Blocking Root Access

files: /etc/pam.d/su /etc/pam.d/sudo 

Adding this setting prohibits root access to members of the wheel group.

`auth       required        pam_wheel.so`

##### Automatic Accounts Blocking

files: /etc/pam.d/logn /etc/pam.d/sshd

`auth       required        pam_tally.so deny=3 lock_time=300`

#### Applying New Password Settings to a User

Changing the `login.defs` file changes the default login and does not apply the settings to existing users. So if you used these settings after a series of users have been created, this can result in users having the default values from the previous file that was present. PAM settings are dynamic and are applied upon authentication, so those settings are already in effect.

##### Viewing Password Information

To view the current password expiry information:


`chage -l <user>`

```bash
<OUTPUT>

Last password change                    : Jul 26, 2018
Password expires                        : never
Password inactive                       : never
Account expires                         : never
Minimum number of days between password change        : 0
Maximum number of days between password change        : 99999
Number of days of warning before password expires     : 7

<OUTPUT>

```

##### Change Expire Date

This sets the maximum amount of days that a password can be in effect.

`chage -M <days> <user>`


##### Change Warn Date


This setting sets the password to warn days to warn the user until the password is expired.


`chage -W <days> <user>`


##### Locking an Account

To immediately lock out an account, you can do that with:

`usermod -L <user>`

To set a specific date to lock out an account, you can do that with:

`chage -E <date> <user>`

##### All Values

You are not required to know every command or review the man page to change the specified user's password settings. You can run the command with no arguments, and it will prompt you for settings.


`chage <user>`

According to `die.net` on manpage 1:

```man
If none of the options are selected, the chage operates interactively, prompting the user with the current fields' values. Enter the new value to change the field, or leave the line blank to use the current value. The current value is displayed between a pair of [ ] marks. 
```

###### Sources:

- ###### <https://www.debian.org/doc/manuals/securing-debian-manual/ch04s11.en.html>
- ###### <https://www.howtoforge.com/linux-chage-command/>
- ###### <https://linux.die.net/man/1/chage>
- ###### <https://www.ssi.gouv.fr/en/guide/configuration-recommendations-of-a-gnulinux-system/>
---

###### [Home](https://mksipe.github.io/mksipe/) | [Privacy Policy](https://mksipe.github.io/mksipe/Privacy)