# Поднять WORDPRESS c помощью docker-compose c доступом к сайту через nginx c сертификатами letsencrypt #

Поднимаем так же базу данных MariaDB с доступом к ней через phpmyadmin

Для получения сертификата необходимо остановить контейнер nginx и запустить:
```
docker compose run --rm -p 80:80 -p 443:443 --entrypoint  "certbot certonly --standalone -d gl-era.com" certbot
```

Для обновления сертификата необходимо остановить контейнер nginx и запустить:
```
docker-compose run --rm -p 80:80 -p 443:443 --entrypoint  "certbot renew --standalone" certbot
```

=======

## Защита nginx и хоста с помощью CrowdSec ##

Сайт документации  проекта: [https://docs.crowdsec.net](https://docs.crowdsec.net)

Защита организована через анализ данных логов nginx и auth.log хоста.

Проверяем, читает ли наша установка CrowdSec журналы должным образом:

```bash
docker compose exec crowdsec cscli metrics
```

Список команд cscli Hub позволяет нам увидеть, какие парсеры и сценарии развернуты:

```bash
docker compose exec crowdsec cscli hub list
```

Доступен веб-интерфейс Metabase с дашбордами умолчанию на https://domain.com:3000/. Написан на java - ресурсоемкий.

Логин по умолчанию: crowdsec@crowdsec.net Пароль: !!Cr0wdS3c_M3t4b4s3??

## Блокировка атак через Bouncer ##

Установка на хосте:
```bash
wget https://github.com/crowdsecurity/cs-firewall-bouncer/releases/download/v0.0.28/crowdsec-firewall-bouncer-linux-amd64.tgz
tar xzf crowdsec-firewall-bouncer-linux-amd64.tgz
cd crowdsec-firewall-bouncer-v0.0.28
sudo ./install.sh
```
Cоздание токена API для нашего bouncer:

```bash
docker compose exec crowdsec cscli bouncers add HostFirewallBouncer
```

Добавляем токен и адрес контейнера (http://172.20.0.6:8080/) в  /etc/crowdsec/bouncers/crowdsec-firewall-bouncer.yaml и перезапускаем службу.

Посмотреть список заблокированных ip-адресов:

```bash
docker compose exec crowdsec cscli decisions list
```

Удалить ip-адрес из списка:

```bash
docker compose exec crowdsec cscli decisions delete -i 1.2.3.4
```

## Защита Wordpress c помощью CrowdSec ##

[https://docs.crowdsec.net/u/bouncers/wordpress/](https://docs.crowdsec.net/u/bouncers/wordpress/)

Устанавливаем плагин в wordpress.

Получаем API для bouncer wordpress:

```bash
docker compose exec crowdsec cscli bouncers add WordpressBouncer
```

Добавляем полученный ключ API в настройки плагина в Bouncer API key.

Local API URL: http://crowdsec:8080

Устанавливаем
```bash
docker compose exec cscli collections install crowdsecurity/wordpress
docker compose exec cscli collections install crowdsecurity/appsec-wordpress
```

## Белый список (whitelist) ip адресов ##

Посмотреть список адресов:

``` bash
docker compose exec crowdsec cat /etc/crowdsec/parsers/s02-enrich/whitelists.yaml
```

Добавить адрес в список:

```bash
docker compose exec crowdsec sh -c "sed '/ip:/a\\    - \"1.2.3.4\"' /etc/crowdsec/parsers/s02-enrich/whitelists.yaml > /tmp/whitelists.yaml && mv /tmp/whitelists.yaml /etc/crowdsec/parsers/s02-enrich/whitelists.yaml"
```
