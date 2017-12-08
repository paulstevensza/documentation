# FreeBSD SSHGuard

Because I'm a moron, SSHGuard wasn't working properly when I set it up on a new instance this morning. Because the best
backend for SSHGuard is not, in fact, `/usr/local/libexec/sshg-fw-null`, which I hadn't amended.

Anyway.

To get going:

`pkg install sshguard`

Enable it in `/etc/rc.conf`:

```
sshguard_enable="YES"
```

Then, add the following to your `/etc/pf.conf`:

```
# SSHGuard config
table <sshguard> persist
block in proto tcp from <sshguard>
```

Finally, change the backend in `/usr/local/etc/sshguard.conf`:

```
#### REQUIRED CONFIGURATION ####
# Full path to backend executable (required, no default)
#BACKEND="/usr/local/libexec/sshg-fw-null"
#BACKEND="/usr/local/libexec/sshg-fw-ipfw"
BACKEND="/usr/local/libexec/sshg-fw-pf"
```

What you're pretty much going is commenting out:

```
#BACKEND="/usr/local/libexec/sshg-fw-null"
```

and uncommenting

```
BACKEND="/usr/local/libexec/sshg-fw-pf"
```

Finally, bounce the SSHGuard service:

```
service sshguard restart
```

Now, you can see who is knocking at your SSH port, and who PF is blocking:

```
sudo pfctl -t sshguard -T show
   94.177.218.11
   125.212.202.52
   159.89.28.90
```

Done and dusted.
