<p style="text-align: center; font-size: 40px;">Hardening SSH</p>

---

##### Metadata

###### Last revised - 10/4/21

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

### Login Defaults

The `/etc/login.defs` file is designed to apply default values to a user when they are not previously def
ined. ***These values are not applied unless a user is created.***

#### Set the Maximum Password Expire

Setting the maximum time for a password to be used enhances security by forcing a user to change their password after logging in after x days.

`PASS_MAX_DAYS 60`

#### Set the Minimum Password Expire

Setting the minimum amount of time until a password can help in case of a compromise and a user's password can still be used, at this point, an administrator would have to change it. 

`PASS_MIN_DAYS 7`

#### Set the Minimum Password Length

Setting a specific length can help make password cracking take longer by brute force, thus, making it more secure.

`PASS_MIN_LEN 12`

#### Set a Password Expire Warning

Warn users upon an upcoming password change.

`PASS_WARN_AGE 5`

#### Using a Safer UMASK

Chaning the usmask chooses the default file permissions which is 077

This should be changed to retain privacy and security.

`UMASK 027` - `UMASK 022` is also preferred

this should also be changed in the /etc/rc.d/rc file as well


### Setting Password Complexity with PAM

You can also specify specific requirements with PAM in order to only allow passwords that have specific characters in them.

You need to have the `libpam-cracklib` package installed in order for this section to work.  

#### Common Password File

The common password file sets the specific requirements in order to allow specific passwords to be used.

`/etc/pam.d/common-password`


```
password    requisite     pam_cracklib.so try_first_pass retry=3
password    sufficient    pam_unix.so md5 shadow nullok try_first_pass use_authtok
password    required      pam_deny.so
```

> It is suggested to backup your pam files. If you don't you may not be able to authenticate.


You need to add the following after `requisite pam_cracklib.so`

`retry=3 minlength=16 lcredit=-1 ucredit=-1 dcredit=-1 ocredit=-1 difok=4`

The file should look like this:

```
password    requisite     pam_cracklib.so try_first_pass retry=3 minlength=16 lcredit=-1 ucredit=-1 dcredit=-1 ocredit=-1 difok=4
password    sufficient    pam_unix.so md5 shadow nullok try_first_pass use_authtok sha512
password    required      pam_deny.so
```

What this addition does makes it so that passwords now are required to be at least 16 caracters long, require a lowercase, uppercase, digit, special character, and no dictionary words within a password. 



###### Sources:

---

###### [Home](https://mksipe.github.io/mksipe/)