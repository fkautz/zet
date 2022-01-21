# CVE-2022-0185

* TLDR 1: Update your kernel!
* TLDR 2: Always drop capabilities or future CVEs will burn you!

A recent vulnerability was discovered where an integer underflow bypasses a bounds check, allowing for unconstrained writes to anywhere in memory.
Skipping most of the details, I discovered in the announcement that `CAP_SYS_ADMIN` is required to exploit it.
Typically, you are protected from these capabilities causing harm out of their namespaces.
However, bugs happen and sometimes something slips through.

From the CVE:

```
Exploitation relies on the CAP_SYS_ADMIN capability; however, the permission only needs to be granted in the current 
namespace. An unprivileged user can use unshare(CLONE_NEWNS|CLONE_NEWUSER) to enter a namespace with the 
CAP_SYS_ADMIN permission, and then proceed with exploitation to root the system.
````

## How to gain capabilities when you have none:

Unshare creates new namespaces and executes a program in that namespace.
These namespaces come with their own set of capabilities.
This works even when the starting process has no capabilities of its own.

There are multiple types of namespaces which can be created by unshare.
These include but are not limited to:

* cgroup
* PID
* user
* time 
* mount

More accurately, it creates a new nanmespace and moves a process into that namespace. The namespaces may also be persistent by binding them to a file. Persistent namespaces may be entered using `nsenter`.

On a user in ubuntu 20.04 LTS, I performed the following:

```sh
vespera@ubuntu-s-1vcpu-1gb-amd-sfo3-01:~$ cat /proc/self/status | grep Cap
CapInh:	0000000000000000
CapPrm:	0000000000000000 # NOTICE THIS IS ALL ZEROS, NO CAPABILITIES!!!
CapEff:	0000000000000000
CapBnd:	0000003fffffffff
CapAmb:	0000000000000000
```

CapInh is inherited capabilities, which we can see are set entirely to 0.

```sh
vespera@ubuntu-s-1vcpu-1gb-amd-sfo3-01:~$ capsh --decode=0000000000000000
0x0000000000000000=
```

We can see above that there are no effective capabilities at all.

```sh
vespera@ubuntu-s-1vcpu-1gb-amd-sfo3-01:~$ unshare -r -m /bin/bash
root@ubuntu-s-1vcpu-1gb-amd-sfo3-01:~#
```

We now have a new `mount` and `user` namespace with the UID and GID set to a superuser.

```sh
root@ubuntu-s-1vcpu-1gb-amd-sfo3-01:~# cat /proc/self/status | grep Cap
CapInh:	0000000000000000
CapPrm:	0000003fffffffff
CapEff:	0000003fffffffff
CapBnd:	0000003fffffffff
CapAmb:	0000000000000000
```

Let's see what capabilities we now have:

```sh
root@ubuntu-s-1vcpu-1gb-amd-sfo3-01:~# capsh --decode=0000003fffffffff
0x0000003fffffffff=cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,cap_audit_read
```

However, despite having these new capabilities, the parent namespace restrictions still typically apply:

```sh
root@ubuntu-s-1vcpu-1gb-amd-sfo3-01:/etc# cat /etc/shadow
cat: /etc/shadow: Permission denied
```

Once you have gained the `CAP_SYS_ADMIN` capability, the exploit in legacy_parse_param becomes reachable and the system may then be exploited.

## Disabling unprivileged user namespace creation

As root (real root):
```sh
sysctl -w kernel.unprivileged_userns_clone=1
```

As the user:
```sh
unshare -m -r /bin/bash
unshare: unshare failed: Operation not permitted
```

## References

* https://seclists.org/oss-sec/2022/q1/55
* https://www.kernel.org/doc/html/latest/userspace-api/unshare.html
* https://www.redhat.com/sysadmin/mount-namespaces
