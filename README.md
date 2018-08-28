# zabbix-ssl_check

https://www.null-byte.org/development/monitoring-ssl-certificates-with-zabbix/
https://blog.mailon.com.ua/%D0%BC%D0%BE%D0%BD%D0%B8%D1%82%D0%BE%D1%80%D0%B8%D0%BD%D0%B3-ssl-%D1%81%D0%B5%D1%80%D1%82%D0%B8%D1%84%D0%B8%D0%BA%D0%B0%D1%82%D0%BE%D0%B2-%D0%B2-zabbix/

> nano /usr/lib/zabbix/externalscripts/ssl_check.sh

#! /bin/sh
SERVER=$1
TIMEOUT=25
RETVAL=0
TIMESTAMP=`echo | date`
if [ -z "$2" ]
then
PORT=443;
else
PORT=$2;
fi
EXPIRE_DATE=`echo | openssl s_client -connect $SERVER:$PORT -servername $SERVER 2>/dev/null | openssl x509 -noout -dates 2>/dev/null | grep notAfter | cut -d'=' -f2`
EXPIRE_SECS=`date -d "${EXPIRE_DATE}" +%s`
EXPIRE_TIME=$(( ${EXPIRE_SECS} - `date +%s` ))
if test $EXPIRE_TIME -lt 0
then
RETVAL=0
else
RETVAL=$(( ${EXPIRE_TIME} / 24 / 3600 ))
fi
 
echo "$TIMESTAMP | $SERVER:$PORT expires in $RETVAL days" >> /usr/lib/zabbix/externalscripts/ssl_check.log
echo ${RETVAL}

> cat /etc/zabbix/zabbix_server.conf | grep externalscripts
# ExternalScripts=${datadir}/zabbix/externalscripts
ExternalScripts=/usr/lib/zabbix/externalscripts

> chmod +x /usr/lib/zabbix/externalscripts/ssl_check.sh

> touch /usr/lib/zabbix/externalscripts/ssl_check.log
> chown zabbix:zabbix /usr/lib/zabbix/externalscripts/ssl_check.log

> systemctl restart zabbix-server
