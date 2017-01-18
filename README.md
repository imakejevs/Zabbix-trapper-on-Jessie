## How-to install Zabbix SNMP trapper on Debian 8 (Jessie)

Run all commands as root or with sudo.

1. Install SNMP trapper daemon and Perl SNMP library from packages:
```
>apt-get update
>apt-get install snmptrapd libsnmp-perl
```

2. Acquire and copy Zabbix trapper Perl script:
```
>cd /tmp
>curl -sLo- https://downloads.sourceforge.net/project/zabbix/ZABBIX%20Latest%20Stable/3.2.3/zabbix-3.2.3.tar.gz | tar xvz zabbix-3.2.3/misc/snmptrap/zabbix_trap* --strip-components=3
>mv zabbix_trap_receiver.pl /usr/bin/
```

3. Edit Zabbix trapper Perl script and adjust trapper file path if needed:
```
>nano /usr/bin/zabbix_trap_receiver.pl

$SNMPTrapperFile = '/tmp/zabbix_traps.tmp';
```
4. Adjust permissions to make it executable:
```
>chmod +x /usr/bin/zabbix_trap_receiver.pl
```

5. Edit SNMP daemon config file to disable it, stop SNMP daemon:
```
>nano /etc/default/snmpd

SNMPDRUN=no
```
```
>service snmpd stop
```

6. Enable SNMP trapper:
```
>nano /etc/default/snmptrapd

TRAPDRUN=yes
```

7. Edit SNMP trapper config file and adjust your community name or disable auth:
```
>nano /etc/snmp/snmptrapd.conf

#authCommunity execute public
disableAuthorization yes
perl do "/usr/bin/zabbix_trap_receiver.pl";
```

8. Start SNMP trapper daemon:
```
>service snmptrapd restart
```

9. Check that SNMP trapper is listening on UDP port 162:
```
>ss -lnup | grep 162

UNCONN     0      0           *:162         *:*      users:(("snmptrapd",pid=2460,fd=7))
```

10. Adjust `/etc/zabbix/zabbix_server.conf` to start SNMP trappers and trapper file path (from step 3) if needed:
```
StartSNMPTrapper=1
SNMPTrapperFile=/tmp/zabbix_traps.tmp
```

11. Reload Zabbix server configuration cache:
```
>zabbix_server -R config_cache_reload
```

12. Send test SNMP trap from your device and check contents of `/tmp/zabbix_traps.tmp` file.
