# FreeBSD Jails

Stuff I know about creating FreeBSD jails in less than 5 minutes. Note that I use `ezjails` because its pretty idiot proof.

`sudo pkg install ezjail`

`sudo ezjail-admin install -p`

Add the following to `/etc/rc.conf`:

```
pf_enable="YES"

# Enable EZJails
ezjail_enable="YES"

# Set up clones interfaces for use with jails
cloned_interfaces="lo1"
ifconfig_lo1="127.0.2.1 netmask 255.255.255.0"
```

then

`sudo service netif cloneup`

`ifconfig` would show the following:

```
lo1: flags=8049<UP,LOOPBACK,RUNNING,MULTICAST> metric 0 mtu 16384
	options=600003<RXCSUM,TXCSUM,RXCSUM_IPV6,TXCSUM_IPV6>
	inet 127.0.2.1 netmask 0xffffff00
	nd6 options=21<PERFORMNUD,AUTO_LINKLOCAL>
	groups: lo
```

Then edit the hell out of `/etc/pf.conf`:

```
# Define interfaces
ext_if="vtnet0"
ext_addr="159.89.28.90"
int_if="lo1"
jail_net = $int_if:network

# Setup for redirects to nginx jail
NGINX="127.0.2.1"
NGINX_TCP_PORTS="{80, 443}"

# Define NAT for jails
nat on $ext_if from $jail_net to any -> $ext_addr port 1024:65535 static-port

# Redirect 80,443 to nginx jail
rdr pass on $ext_if inet proto tcp to port $NGINX_TCP_PORTS -> $NGINX
```

Then create and access a new jail (you only really need the last two commands):

```
ezjail-admin create nginx 127.0.2.1
ezjail-admin list
ezjail-admin start nginx
ezjail-admin list
cp /etc/resolv.conf /usr/jails/nginx/etc
ezjail-admin console nginx
```

Inside of your jail:

```
pkg install nginx
echo 'nginx_enable="YES"' > /etc/rc.conf.d/nginx
service nginx start
```

Have a quick look to see if you're getting the default Nginx landing page.

Add the following to pf's config:

```
# Drop everything by default
block all

# Allow traffic to jails to be translated
pass from { lo0, $jail_net } to any keep state

# Allow SSH to the host
pass in inet proto tcp to $ext_if port ssh

# Allow outbound traffic
pass out all keep state
```

Check PF's rules:

`sudo pfctl -nf /etc/pf.conf`

If it doesn't flag any errors,

`sudo pfctl -f /etc/pf.conf`

Profit.
