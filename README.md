# Codeless-checks-knowledge-base
How to get Nagios / Icinga to communicate with various daemons without additional code. 

In many cases, check_tcp is used. This is found in the *monitoring-plugins-basic* package (Debian) or *nagios-plugins-tcp* package (RHEL).

In other cases, we can run a third-party binary directly, as long as it returns appropriate exit status on success and failure. Where it doesn't, a wrapper is needed, in which case it is beyond scope of this article.

# Redis
```
/usr/lib64/nagios/plugins/check_tcp -H HOST -p 6379 -E -s 'PING\n' -e PONG
```

# Memcached
```
/usr/lib64/nagios/plugins/check_tcp -H HOST -p 11211 -E -s 'stats\n' -e 'STAT uptime'
```

# InfluxDB
```
/usr/bin/influx --execute "show measurements" -database XXX -HOST XXX -username XXX -password XXX
```

# Asterisk
```
/usr/bin/sudo /usr/sbin/asterisk -r -s /var/run/asterisk/asterisk.ctl -x "core show uptime"
```

# Munin-node
```
/usr/lib64/nagios/plugins/check_tcp -H localhost -p 4949 -e '# munin node at '
```
# Qpidd
```
/usr/lib64/nagios/plugins/check_tcp -H localhost -p 5672 -s 'icinga:icinga' -e AMQP
```
# Other tips
* Use `check_smtp` for amavis. The package is *monitoring-plugins-basic* on Debian and *nagios-plugins-smtp* on RHEL.
* Use `check_http` for nodejs, and java application servers. The package is *monitoring-plugins-basic* on Debian and *nagios-plugins-http* on RHEL.
* When implementing `check_tcp`, be familiar with the `-s` and `-e` switches. Your test is much more robust if it tests for the expected behaviour, not just a listening socket.
  * Also keep in mind this test can connect to UNIX sockets when run on the remote agent.
* When implementing check_http, use `-s` to search for expected content on the page.
  * Also use `-v -f follow` to discover the direct URI. Then in production, pass the URI to the -u switch (e.g. `-u '/dashboard/Login.php'`).

# References
https://www.netkiller.cn/monitoring/nagios/plugins.html
https://influxdbcom.readthedocs.io/en/latest/content/docs/v0.9/query_language/schema_exploration/
https://www.influxdata.com/blog/how-to-use-the-show-stats-command-and-the-_internal-database-to-monitor-influxdb/
