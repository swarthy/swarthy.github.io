---
title: certbot renew hooks
date: 2020-02-10 14:31:07
tags: administration certbot
---

Для случая, когда сертификаты обновляются методом webroot и необходимо вручную перезагрузить конфиг nginx'а.

`/etc/letsencrypt/renewal-hooks/deploy/nginx`

```bash
#!/bin/sh
DATE=`date '+%Y-%m-%d %H:%M:%S'`
echo $DATE "Deploy hook ran." >> /var/log/certbot-renew.log

/bin/systemctl reload nginx >> /var/log/certbot-renew.log

```

> Важно!
>
> Не трогай /etc/cron.d/certbot, он не работает, когда в системе запущен systemd

Также не помешает настроить ротацию лога
`/etc/logrotate.d/certbot-renew`

```
/var/log/certbot-renew.log {
        weekly
        missingok
        rotate 12
        compress
        notifempty
	      su root root
}
```
