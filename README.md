# codeless-checks-knowledge-base
How to get Nagios / Icinga to communicate with various daemons without additional code. 

In some cases, check_tcp is used. This is found in the monitoring-plugins-basic package (debian) or nagios-plugins-tcp package (Red Hat).

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

# References
https://www.netkiller.cn/monitoring/nagios/plugins.html
https://influxdbcom.readthedocs.io/en/latest/content/docs/v0.9/query_language/schema_exploration/
https://www.influxdata.com/blog/how-to-use-the-show-stats-command-and-the-_internal-database-to-monitor-influxdb/
