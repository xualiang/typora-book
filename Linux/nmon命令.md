/opt/nmon -s 5 -c 1440 -f -m /opt/nmon_log/

 

Vim /var/spool/cron/root

0 */2 * * * /opt/nmon -s 5 -c 1440 -f -m /opt/nmon_log/

 

service crond restart