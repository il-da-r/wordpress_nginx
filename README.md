#Поднять WordpPress c помощью docker-compose c доступом к сайту через nginx c сертификатами letsencrypt#

Поднимаем так же базу данных MariaDB с доступом к ней через phpmyadmin\

Для получения сертификата необходимо остановить контейнер nginx и запустить:\
```
docker compose run --rm -p 80:80 -p 443:443 --entrypoint  "certbot certonly --standalone -d gl-era.com" certbot
```

Для обновления сертификата необходимо остановить контейнер nginx и запустить:\
```
docker-compose run --rm -p 80:80 -p 443:443 --entrypoint  "certbot renew --standalone" certbot
```
=======
