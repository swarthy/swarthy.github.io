---
title: Обновляем PostgreSQL до 12 версии
date: 2020-01-13 19:05:54
tags: administration, postgresql
---

Для удобства следует открыть 2 терминала: в одном залогиниться под `root`, во втором под `postgres`

# Устанавливаем 12 версию

```bash root
apt update
apt install install postgresql-12 postgresql-server-dev-12
```

Так как уже установлена 11, в конфигах 12ой будет настроен порт 5433 вместо 5432

# Переносим настройки

```bash root
diff /etc/postgresql/11/main/postgresql.conf /etc/postgresql/12/main/postgresql.conf
diff /etc/postgresql/11/main/pg_hba.conf /etc/postgresql/12/main/pg_hba.conf

nano /etc/postgresql/12/main/postgresql.conf
# обязательно поменять порт с 5433 на 5432

nano /etc/postgresql/12/main/postgresql.conf
# обязательно поменять порт с 5432 на 5431, например
```

# Останавливаем сервис

```bash
systemctl stop postgresql.service
```

# Переносим данные

`pg_upgrade` запускаем от имени пользователя `postgres`, также `postgres` должен иметь права на запись в cwd

## Создаем папку с доступом для postgres

```bash root
mkdir pglog
chown postgres:postgres pglog
```

## Проверка (не изменяет данные, см --check)

```bash postgres
cd pglog
/usr/lib/postgresql/12/bin/pg_upgrade \
  --old-datadir=/var/lib/postgresql/11/main \
  --new-datadir=/var/lib/postgresql/12/main \
  --old-bindir=/usr/lib/postgresql/11/bin \
  --new-bindir=/usr/lib/postgresql/12/bin \
  --old-options '-c config_file=/etc/postgresql/11/main/postgresql.conf' \
  --new-options '-c config_file=/etc/postgresql/12/main/postgresql.conf' \
  --check
```

## Миграция

Может занять некоторое время (для 10гб базы минут 10), но все равно быстрее чем pg_dump + psql

```bash postgres
/usr/lib/postgresql/12/bin/pg_upgrade \
  --old-datadir=/var/lib/postgresql/11/main \
  --new-datadir=/var/lib/postgresql/12/main \
  --old-bindir=/usr/lib/postgresql/11/bin \
  --new-bindir=/usr/lib/postgresql/12/bin \
  --old-options '-c config_file=/etc/postgresql/11/main/postgresql.conf' \
  --new-options '-c config_file=/etc/postgresql/12/main/postgresql.conf'
```

# Запускаем сервис

```bash root
systemctl start postgresql.service
```

# Убеждаемся, что всё работает

```bash postgres
psql -c "SELECT version();"
```

# Удаляем 11 версию

```bash root
apt list --installed | grep postgresql
apt remove postgresql-11 postgresql-server-dev-11 postgresql-client-11
```

`pg_upgrade` генерирует delete_old_cluster.sh в `pglog`
Если всё прошло успешно, то он вызвался и удалил данные кластера 11 версии, но если что-то пошло не так, придется удалять руками:

```bash
# данные
rm -rf '/var/lib/postgresql/11/main'

# конфиги (delete_old_cluster не удаляет их)
rm -rf '/etc/postgresql/11'
```
