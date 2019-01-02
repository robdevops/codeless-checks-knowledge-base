# Codeless checks knowledge base
How to get Nagios / Icinga to communicate with various daemons without additional code. 

In many cases, these commands use `check_tcp` from the *monitoring-plugins-basic* package (Debian), or the *nagios-plugins-tcp* package (RHEL).

In other cases, we can run the built-in management tools of the target service directly, as long as it returns appropriate exit status on success and failure. Where it doesn't, a wrapper is needed, and is therefore beyond the scope of this article.

# Asterisk
```
/usr/bin/sudo /usr/sbin/asterisk -r -s /var/run/asterisk/asterisk.ctl -x "core show uptime"
```

# DHCPD
Okay, this one isn't super creative, but it may not be obvious that dhcpd has this built-in, since you must define both `-i lo` and `-s 127.0.0.1` to get a response. Anyway, it sure beats junk entries into dhcpd.conf.
```
/usr/bin/sudo /usr/lib64/nagios/plugins/check_dhcp -t 1 -i lo -u -s 127.0.0.1 -r 0.0.0.0
```

# FreeRADIUS
1. Install the *freeradius-utils* package.
1. Create the test payload: `echo 'Message-Authenticator = 0x00' > /usr/local/etc/radclient.status.txt`
1. Run the test:
```
/usr/bin/radclient localhost:18121 status adminsecret -f /usr/local/etc/radclient.status.txt
```

# InfluxDB
```
/usr/bin/influx --execute "show measurements" -database XXX -HOST XXX -username XXX -password XXX
```

# Memcached
```
/usr/lib64/nagios/plugins/check_tcp -H HOST -p 11211 -E -s 'version\n' -e 'VERSION'
```

# Munin-node
```
/usr/lib64/nagios/plugins/check_tcp -H localhost -p 4949 -e '# munin node at '
```

# MySQL / MariaDB
The packaged `check_mysql_query` is not designed to query for a string. It requires numerical output from the query. However, given that Nagios tests are in essence a "pass" or "fail", we can use an SQL `IF` statement to substitute the presence or absence of any value with a number. In this case, if `variable_value='Synced'`, we return `1`, else we return `0`:
```
SELECT IF (variable_value='Synced',1,0) FROM information_schema.global_status WHERE variable_name='wsrep_local_state_comment';
```
Putting that into the `check_mysql_query`, we use `-w 1:1` to alert if the result is not 1. Also note the single quotes require escaping:
```
/usr/lib64/nagios/plugins/check_mysql_query -H localhost -d portal -u icinga -p password -q "SELECT IF (variable_value=\'Synced\',1,0) FROM information_schema.global_status WHERE variable_name=\'wsrep_local_state_comment\' -w 1:1 
```

# Qpidd
```
/usr/lib64/nagios/plugins/check_tcp -H localhost -p 5672 -s 'icinga:icinga' -e AMQP
```

# Redis
```
/usr/lib64/nagios/plugins/check_tcp -H HOST -p 6379 -E -s 'PING\n' -e PONG
```


# Other tips
* Use `check_smtp` for amavis. The package is *monitoring-plugins-basic* on Debian and *nagios-plugins-smtp* on RHEL.
* Use `check_http` for nodejs, and java application servers. The package is *monitoring-plugins-basic* on Debian and *nagios-plugins-http* on RHEL.
* When implementing `check_tcp`, be familiar with the `-s` and `-e` switches. Your test is much more robust if it tests for the expected behaviour, not just a listening socket.
  * Also keep in mind this test can connect to UNIX sockets when run on the remote agent.
* When implementing `check_http`, use `-s` to search for expected content on the page.
  * Also use `-v` `-f follow` to discover the final URI after all redirects. In production, pass this final URI to the `-u` switch (e.g. `-u '/dashboard/Login.php'`) to increase efficiency.

# References
https://www.netkiller.cn/monitoring/nagios/plugins.html
https://influxdbcom.readthedocs.io/en/latest/content/docs/v0.9/query_language/schema_exploration/
https://www.influxdata.com/blog/how-to-use-the-show-stats-command-and-the-_internal-database-to-monitor-influxdb/
