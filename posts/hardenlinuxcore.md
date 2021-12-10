<p style="text-align: center; font-size: 40px;">Hardening the Linux Kernel</p>

---

##### Metadata

###### Last revised - 10/28/21

###### Author       - Mason Sipe

---

## Preamble

This post is about hardening the core components of the Linux kernel.
The article will cover a majority of commonly misconfigured settings in different system files for daily usage of the Linux system. This post will also help with restricting available information that may be valuable to a perpetrator.

---

## Background

The system control file is a file that directly modifies the settings of the running Linux kernel. The sysctl.conf file is designed to override the default kernel values. These values are essential as they change network settings, kernel settings, security settings, limits, and other potentially sensitive or critical settings. 

## Table of Contents

- [Hardening the Linux Kernel](#table-of-contents)
    - [Disabling Core Dumps](#disabling-core-dumps)
    - [Wabbit Mitiation](#preventing-a-fork-bomb-rabbit-virus-wabbit)
    - [SystemCtl](#systemctl)
        - [Packet Forwarding](#packet-forwarding)
        - [No Routing](#no-routing)
        - [SysRQ](#sysrq)
        - [Core Dumps Append PID](#core-dumps-append-pid)
        - [Syn-Flood Protection](#syn-flood-protection)
        - [Disable IPv4 Routing](#disable-ipv4-routing)
        - [Deny Secure Routing (we are not a router)](#deny-secure-routing-we-are-not-a-router)
        - [We do Not Redirect Traffic](#we-do-not-redirect-traffic)
        - [Log Impossible Addresses](#log-impossible-addresses)
        - [Ignore ICMP Requests and Broadcasts](#ignore-icmp-requests-and-broadcasts)
        - [Prevent Against Common Syn Flood Attacks](#prevent-against-common-syn-flood-attacks)
        - [Source Validation by Reverse Path](#source-validation-by-reverse-path)
        - [Default Source Route Verification](#default-source-route-verification)
        - [This is a Host](#this-is-a-host)
        - [Accept Router's Preference in Router Advertisements](#accept-routers-preference-in-router-advertisements)
        - [Learn Prefix in Routers Advertisement](#learn-prefix-in-routers-advertisement)
        - [Accept Hop Limit](#accept-hop-limit)
        - [Assign Global Unicast Address on an Interface](#assign-global-unicast-address-on-an-interface)
        - [Neighbor Solicitation per Address](#neighbor-solicitation-per-address)
        - [Enable ExecShield Protection](#enable-execshield-protection)
        - [Optimize TCP and Memory](#optimize-tcp-and-memory)
        - [Increase Auto Tuning TCP Buffer](#increase-auto-tuning-tcp-buffer)
        - [Increase Filesystem Descriptor Limit](#increase-filesystem-descriptor-limit)
        - [Allow for More PIDs](#allow-for-more-pids)
        - [Increase System IP Port Limit](#increase-system-ip-port-limit)
        - [Fix for RFC 1337](#fix-for-rfc-1337)
        - [Reboot After Kernel Panic](#reboot-after-kernel-panic)
        - [Ignore Bad ICMP Errors](#ignore-bad-icmp-errors)
        - [Safely Use Links](#safely-use-links)
        - [Prevent Autoloading of ldiscs](#prevent-autoloading-of-ldiscs)
        - [Protect First in First Out](#protect-first-in-first-out)
        - [Restrict Exposed Kernel Addresses](#restrict-exposed-kernel-addresses)
        - [CVE-2020-8835](#cve-2020-8835)
        - [Protect Against Ptrace Processes](#protect-against-ptrace-processes)
        - [Harden Just In Time Compiler](#harden-just-in-time-compiler)
- [Sources](#sources)

### Disabling Core Dumps

According to Vivek Gite: "Core dumps created for diagnosing and debugging errors in Linux apps. They are also known as memory dump, crash dump, system dump, or ABEND dump. However, core dumps may contain sensitive info—for example, passwords, user data such as PAN, SSN, or encryption keys. Hence, we must disable them on production Linux servers."

From this, you can infer that the information stored in core dumps may be susceptible if not kept securely or stored. 

To disable core dumps, you will need to add this line in `/etc/security/limits.conf`

```
* hard core 0
* soft core 0
```

Then, in `/etc/sysctl.conf` add the following line:
```
fs.suid_dumpable = 0
kernel.core_pattern=|/bin/false

``` 


### Preventing a Fork Bomb (rabbit virus, wabbit)

A fork bomb is of what follows: `:(){ :|:& };:`. (DO NOT RUN THIS IN A SHELL)

This process continuously creates child processes that infinitely spawn their processes, depleting your system of resources. 


<img align="center" width="900" height="800" src="https://raw.githubusercontent.com/mksipe/mksipe/gh-pages/_layouts/assets/Wabbit.png">

A detailed explanation of how fork bombs work is in [this](https://www.cyberciti.biz/faq/understanding-bash-fork-bomb/) article.

To prevent fork bombs, you need to add this line to: `/etc/security/limits.conf`


```bash
<$USER> hard noproc 300
@student hard nproc 50
@faculty soft nproc 100
@pusers hard nproc 200
```

### Systemctl 

The following settings are stored in `/etc/sysctl.conf`


#### Packet Forwarding

```bash
# Controls IP packet forwarding
net.ipv4.ip_forward = 0
```

#### No Routing

```bash
# Do not accept source routing
net.ipv4.conf.default.accept_source_route = 0
```

#### SysRQ

```bash
# Controls the System Request debugging functionality of the kernel
kernel.sysrq = 0
```
#### Core Dumps Append PID

```bash
# Controls whether core dumps will append the PID to the core filename
# Useful for debugging multi-threaded applications
kernel.core_uses_pid = 1
```

#### Syn-Flood Protection

```bash
# Controls the use of TCP syncookies
# Turn on SYN-flood protections
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_synack_retries = 5
```

#### Disable IPv4 Routing

```bash
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
```

#### Deny Secure Routing (we are not a router)

```bash
net.ipv4.conf.all.accept_source_route = 0
```


#### We do Not Redirect Traffic

```bash
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.secure_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv6.conf.default.accept_redirects = 0
```

#### Log Impossible Addresses

```bash
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.default.secure_redirects = 0
```

#### Ignore ICMP Requests and Broadcasts

```bash
net.ipv4.icmp_echo_ignore_broadcasts = 1
```

#### Prevent Against Common Syn Flood Attacks

```bash
net.ipv4.tcp_syncookies = 1
```

#### Source Validation by Reverse Path

```bash
net.ipv4.conf.all.rp_filter = 1
```

#### Default Source Route Verification

```bash
net.ipv4.conf.default.rp_filter = 1 
```


#### This is a Host

```bash
net.ipv6.conf.default.router_solicitations = 0
```

#### Accept Router's Preference in Router Advertisements

```bash
net.ipv6.conf.default.accept_ra_rtr_pref = 0
```


#### Learn Prefix in Routers Advertisement

```bash
net.ipv6.conf.default.accept_ra_pinfo = 0
```


#### Accept Hop Limit 

```bash
net.ipv6.conf.default.accept_ra_defrtr = 0
```


#### Assign Global Unicast Address on an Interface

```bash
net.ipv6.conf.default.autoconf = 0
```


#### Neighbor Solicitation per Address

```bash
net.ipv6.conf.default.dad_transmits = 0
```


#### Enable ExecShield Protection

```bash
kernel.randomize_va_space=2
kernel.exec-shield = 2
```


#### Optimize TCP and Memory

```bash
net.ipv4.tcp_rmem = 4096 87380 8388608
net.ipv4.tcp_wmem = 4096 87380 8388608
```
#### Increase Auto Tuning TCP Buffer

```bash
net.core.rmem_max = 8388608
net.core.wmem_max = 8388608
net.core.netdev_max_backlog = 5000
net.ipv4.tcp_window_scaling = 1
```

#### Increase Filesystem Descriptor Limit

```bash
fs.file-max = 65535
```

#### Allow for More PIDs

```bash
kernel.pid_max = 65536
```

#### Increase System IP Port Limit

```bash
net.ipv4.ip_local_port_range = 2000 65000
```

#### Fix for RFC 1337

Prevents TIME-WAIT assasinations. 

```bash
net.ipv4.tcp_rfc1337=1
```


#### Reboot After Kernel Panic

```bash
kernel.panic=10
```



#### Ignore Bad ICMP Errors

```bash
net.ipv4.icmp_ignore_bogus_error_responses=1
```

#### Safely Use Links 

```bash
fs.protected_hardlinks=1
fs.protected_symlinks=1
```

#### Prevent Autoloading of ldiscs

The kernel, by default, will load any module of any line discipline that is requested. This is not one of the safest settings to have enabled. Systemctl can disable this feature with:

```bash
dev.tty.ldisc_autoload = 0
```

#### Protect First in First Out

Data is passed through the kernel before it is written to the filesystem. This will help prevent arbitrary code from being exploited.

```bash
fs.protected_fifos = 2
```
#### Restrict Exposed Kernel Addresses

According to kernel.org: 
```"
When kptr_restrict is set to 0 (the default) the address is hashed before
printing. (This is the equivalent to %p.)

When kptr_restrict is set to (1), kernel pointers printed using the %pK
format specifier will be replaced with 0's unless the user has CAP_SYSLOG
and effective user and group ids are equal to the real ids. This is
because %pK checks are done at read() time rather than open() time, so
if permissions are elevated between the open() and the read() (e.g via
a setuid binary) then %pK will not leak kernel pointers to unprivileged
users. Note, this is a temporary solution only. The correct long-term
solution is to do the permission checks at open() time. Consider removing
world read permissions from files that use %pK, and using dmesg_restrict
to protect against uses of %pK in dmesg(8) if leaking kernel pointer
values to unprivileged users is a concern.

When kptr_restrict is set to (2), kernel pointers printed using
%pK will be replaced with 0's regardless of privileges."
```
To apply the most secure settings:

```bash
kernel.kptr_restrict = 2
```

#### CVE-2020-8835 

According to mitre cve: 

```
In the Linux kernel 5.5.0 and newer, the bpf verifier (kernel/bpf/verifier.c) did not properly restrict the register bounds for 32-bit operations, leading to out-of-bounds reads and writes in kernel memory. The vulnerability also affects the Linux 5.4 stable series, starting with v5.4.7, as the introducing commit was backported to that branch. This vulnerability was fixed in 5.6.1, 5.5.14, and 5.4.29. (issue is aka ZDI-CAN-10780) 
```

To fix this issue:

```bash
kernel.unprivileged_bpf_disabled = 1
```

#### Protect Against Ptrace Processes

According to Linux-audit:

```
Servers

If your system is running in the DMZ and processes high sensitive data, there is usually no reason to allow ptrace at all. Best is to disable it completely (kernel.Yama.ptrace_scope = 3).

For servers in general, you might want to apply rule, or choose a slightly less restrictive value (2 or 1).
Desktops

On desktop systems where you are the only user, can have a less restricted option (2, 1 or even disabled).

The Yama LSM is also used in Google Chrome, as can been seen in the related screenshot. [I will not include this screenshot. see sources for the full article.]

```

```bash
kernel.yama.ptrace_scope = 1 2 3
```
#### Harden Just In Time Compiler

According to kernel.org:

```
bpf_jit_harden
--------------

This enables hardening for the BPF JIT compiler. Supported are eBPF
JIT backends. Enabling hardening trades off performance, but can
mitigate JIT spraying.
Values :
	0 - disable JIT hardening (default value)
	1 - enable JIT hardening for unprivileged users only
	2 - enable JIT hardening for all users

```

The safest setting for this kernel setting would be:
```bash
net.core.bpf_jit_harden = 2
```

###### Sources:

###### - <https://www.cyberciti.biz/faq/disable-core-dumps-in-linux-with-systemd-sysctl/>
###### - <https://www.cyberciti.biz/faq/linux-kernel-etcsysctl-conf-security-hardening/>
###### - <https://www.cyberciti.biz/tips/linux-limiting-user-process.html>
###### - <https://www.kernel.org/doc/Documentation/sysctl/kernel.txt>
###### - <https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-8835>
###### - <https://linux-audit.com/protect-ptrace-processes-kernel-yama-ptrace_scope/>
---

###### [Home](https://mksipe.github.io/mksipe/) | [Privacy Policy](https://mksipe.github.io/mksipe/Privacy)
