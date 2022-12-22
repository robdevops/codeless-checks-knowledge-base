# Config checkers
It's a good idea to sanity check your config changes before attempting to reload/restart a service.
The following commands return non-zero on failure, so can be used in scripting, or in orchestration tools, such as Ansible's ''validate:'' directive.
Or interactively, you could string them together like:
```
checkcommand && reloadcommand
```

## Config
### Apache
```
apachectl configtest
```

### BIND
```
named-checkconf
```

```
named-checkzone opticomm.net.au /var/named/opticomm.net.au.zone
```

Debian/Cumulus Interfaces file:
```
ifreload -as
```

### dhcpd
```
dhcpd -t -cf /etc/dhcp/dhcpd.conf
```

### Icinga2
```
icinga2 daemon -C
```

### IPTables
```
iptables-restore --test /etc/sysconfig/iptables
```

### Keepalived
```
keepalived -t -f /etc/keepalived/keepalived.conf
```

### mysqld
```
mysqld --verbose --help 1>/dev/null
```

### netplan
```
sudo netplan try --timeout 0 && sudo netplan apply
```

### nginx
```
sudo nginx -tc /etc/nginx/nginx.conf
```

### OpenSSH
```
sshd -t
```

### OpenVPN
```
openvpn --config /path/to/server.conf # runs a second instance in foreground
```

### Postfix
```
postfix check
```

### PHP ini
```
<pre>php -r ''</pre>
```

### PHP-FPM
```
sudo php-fpm -t
```

### FreeRADIUS
```
radiusd -C # rhel
```

```
freeradius -C # debian
```

### Squid
```
squid -k check
```

### Sudo
```
/usr/sbin/visudo -c
```


## Code
### bash
```
bash -n script.sh
```

### perl
```
perl -c script.pl
```

### python
```
python -m py_compile script.py
```

### php
```
php -l script.php
```
