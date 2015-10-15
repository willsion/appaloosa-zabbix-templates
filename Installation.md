# Introduction #

Installing and configuring the MySQL templates for Zabbix is fairly straight forward.

**NOTE:** Until there's a tarball release, you need to clone this project with mercurial.

```
hg clone https://appaloosa-zabbix-templates.googlecode.com/hg/ appaloosa-zabbix-templates
cd appaloosa-zabbix-templates
perl tools/gen_template.pl defs/mysql.pl mysql.xml
```

Will generate the template. After that, you need to get the `ss_get_mysql_stats.php` cacti plugin
from http://code.google.com/p/mysql-cacti-templates/ and configure the agent.
There's a template agent configuration file under `conf`, which needs to be updated with the path
to the PHP script on your database server.

You can do something like this:

```
sed -e 's|$ZABBIX_AGENT_PATH|/usr/local/zabbix/plugins|' conf/mysql_agentd.conf > mysql.conf
scp ss_get_mysql_stats.php mydatabase:/usr/local/zabbix/plugins/
scp mysql.conf mydatabase:/usr/local/zabbix/agent.d/
```

Then place that somewhere your zabbix agent can read it and add to your zabbix\_agentd.conf
```
Include=/usr/local/zabbix/agent.d/
```

Finally, import `mysql.xml` into Zabbix, and associate `Template_MySQL` with each database server.

# Configure MySQL #

A dedicated MySQL user should be created for this plugin. Something like the below will work great:
```
GRANT SELECT, SUPER, PROCESS ON *.* TO 'zabbix'@'localhost' identified BY 'some_password';
```

Both `SUPER` and `PROCESS` are required to collect all statistics from MySQL.