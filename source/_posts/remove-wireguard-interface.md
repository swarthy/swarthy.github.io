---
title: 'wireguard: удаление интерфейса'
date: 2020-10-22 17:01:48
tags:
---

Добавить новый интерфейс с помощью `wg-quick` легко. Потушить интерфейс тоже легко. Удалять интерфейс `wg-quick` почему-то не умеет.

Собственно, пример удаления косячно настроенного wg1:

```sh
sudo systemctl stop wg-quick@wg1
sudo systemctl disable wg-quick@wg1.service
sudo rm -i /etc/systemd/system/wg-quick@wg1*
sudo systemctl daemon-reload
sudo systemctl reset-failed
```
