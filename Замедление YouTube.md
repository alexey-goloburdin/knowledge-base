Aeza with marzban template. Clients: hiddify next, streisand. Control panel with ssh-tunnel:

```bash
ssh -L 8000:localhost:8000 www@SERVER_IP
```

And open http://localhost:8000/dashboard/#/login.

Passwords on the server in `~/marzban.txt` file.

Settings, replace YOUR_DOMAIN with your domain:)

```bash
sudo apt install snapd -y
sudo snap install core
sudo snap refresh core
sudo snap install certbot --classic
sudo certbot certonly --standalone -d YOUR_DOMAIN --deploy-hook "cp /etc/letsencrypt/live/YOUR_DOMAIN/privkey.pem /var/lib/marzban/certs/key.pem && cp /etc/letsencrypt/live/YOUR_DOMAIN/fullchain.pem /var/lib/marzban/certs/fullchain.pem"

sudo cp /etc/letsencrypt/live/YOUR_DOMAIN/privkey.pem /var/lib/marzban/certs/key.pem
sudo cp /etc/letsencrypt/live/YOUR_DOMAIN/fullchain.pem /var/lib/marzban/certs/fullchain.pem

sudo vim /opt/marzban/.env

	UVICORN_HOST = "YOUR_DOMAIN"
	UVICORN_PORT = 8000
	ALLOWED_ORIGINS=https://YOUR_DOMAIN:8000,https://YOUR_DOMAIN
	
	UVICORN_SSL_CERTFILE = "/var/lib/marzban/certs/fullchain.pem"
	UVICORN_SSL_KEYFILE = "/var/lib/marzban/certs/key.pem"


sudo marzban restart

# if you need
sudo docker exec -ti marzban_marzban_1 bash

sudo crontab -e
0 3 * * * certbot renew --deploy-hook "cp /etc/letsencrypt/live/YOUR_DOMAIN/privkey.pem /var/lib/marzban/certs/key.pem && cp /etc/letsencrypt/live/YOUR_DOMAIN/fullchain.pem /var/lib/marzban/certs/fullchain.pem"
```

Dashboard settings:

1. Добавить протокол Vless TCP Reality в панель Marzban
2. Сменить Server Names ( по умолчанию установлен example.com)
3. Сменить Dest ( по умолчанию установлен example.com:443)
4. Смена ключа шифрования и параметра shortIds
5. Фикс ошибки Timeout при подключении через приложение Hiddify
6. Создание пользователя

Наблюдаем что в блоке "Inbounds" только Shadowsocks TCP, **добавим Vless TCP Reality в начало блока после квадратной скобки**

```
{
      "tag": "VLESS TCP REALITY",
      "listen": "0.0.0.0",
      "port": 443,
      "protocol": "vless",
      "settings": {
        "clients": [],
        "decryption": "none"
      },
      "streamSettings": {
        "network": "tcp",
        "tcpSettings": {},
        "security": "reality",
        "realitySettings": {
          "show": false,
          "dest": "заменить:443",
          "xver": 0,
          "serverNames": [
            "заменить"
          ],
          "privateKey": "заменить",
          "shortIds": [
            "заменить"
          ]
        }
      },
      "sniffing": {
        "enabled": true,
        "destOverride": [
          "http",
          "tls",
          "quic"
        ]
      }
    },
```

**После добавления протокола Vless TCP Reality требуется изменить значения:** 1. ServerNames 2. Dest 3. Privatekey 4. shortIds.

Подбор Server Name и Dest рекомендуем выполнять командой **ping** Чем ниже пинг до определенного сайта, тем меньше задержка при работающем соединении Необходимо выбирать надежные сайты, если сайт будет недоступен, соединения не будет. **Обязательно выбирайте не популярные но надежные сайты в качестве** **Server Name и Dest для вашей конфигурации Vless TCP Reality.**

**Требования к сайту: 1. TLS 1.3 2. HTTP/2 3. Не должен находиться за CDN сервисом**

```
"dest:" "somesite.ru:443",
"serverNames": ["somesite.ru"]
```

Сгенерируем новые ключи шифрования и параметр shortIds в командной строке вашего сервера командами (на сервере):

```bash
docker exec marzban_marzban_1 xray x25519

# or
docker exec marzban-marzban-1 xray x25519
openssl rand -hex 8
```
Нам необходимо скопировать параметры **Private key** и **shortIds** после чего заменить их в панели.

После внесения изменений, необходимо **сохранить изменения, перезагрузить ядро и перезагрузить страницу в браузере**. На данном моменте Vless TCP Reality настроен.

Нам осталось убрать ошибку "Таймаут" при подключении через программу Hiddify

Возвращаемся в конфигурацию панели и находим пункт "Routing"

Вставляем следующее правило в самое начало блока rules после квадратной скобки

```
{
        "outboundTag": "DIRECT",
        "domain": [
          "full:cp.cloudflare.com",
          "domain:msftconnecttest.com",
          "domain:msftncsi.com",
          "domain:connectivitycheck.gstatic.com",
          "domain:captive.apple.com",
          "full:detectportal.firefox.com",
          "domain:networkcheck.kde.org",
          "full:*.gstatic.com"
        ],
        "type": "field"
      },
```

После внесения изменений, необходимо их **сохранить** и **перезагрузить ядро**.

Создадим пользователя в панели Marzban и выберем протокол Vless TCP Reality.

Копируем ключ подключения к нашему серверу данным образом и переходим в третий пункт **"Подключение ключа в клиент VLESS"**.

### Hiddify next settings

Если после подключения к вашему серверу происходят ошибки, рекомендуем произвести следующие настройки.

Отключим Регион "Россия" и включим режим VPN ( по умолчанию установлен режим Системный прокси). Для включения режима VPN необходимо установить запуск приложения от имени администратора в свойствах ярлыка. После данного действия, перезапускаем приложение и включаем режим VPN.

---

https://github.com/XTLS/Xray-core/discussions/3518

https://startpage.com, Visit in Anonymous View

https://habr.com/ru/articles/799751/ — обзорная статья

![[статья про xray.pdf]]


>[!info] Коммент [отсюда](https://habr.com/ru/articles/832678/):
>
>**Готовые средства обхода.** Ну и самое вкусненькое. Я лично написал [своё решение под линукс](https://github.com/Waujito/youtubeUnblock), которое направлено только на ютуб. Также для Windows существует [GoodbyeDPI от ValdikSS](https://github.com/ValdikSS/GoodbyeDPI), под линукс еще есть [zapret](https://github.com/bol-van/zapret). Существует [ByeDPI](https://github.com/hufrea/byedpi), который работает как прокси (Windows/Linux). Также есть [версия ByeDPI под андройд](https://github.com/dovecoteescapee/ByeDPIAndroid), работает как "фейковый впн". Советую прочитать [подробный комментарий от ValdikSS](https://github.com/yt-dlp/yt-dlp/issues/10443#issuecomment-2248940967) о том, как использовать эти средства. Если есть желание погрузиться глубже в эту тему, вот тут можно посмотреть подробнее: [https://ntc.party/t/замедление-youtube-в-россии/8055/](https://ntc.party/t/%D0%B7%D0%B0%D0%BC%D0%B5%D0%B4%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5-youtube-%D0%B2-%D1%80%D0%BE%D1%81%D1%81%D0%B8%D0%B8/8055/) and [https://ntc.party/t/обсуждение-замедление-youtube-в-россии/8074/](https://ntc.party/t/%D0%BE%D0%B1%D1%81%D1%83%D0%B6%D0%B4%D0%B5%D0%BD%D0%B8%D0%B5-%D0%B7%D0%B0%D0%BC%D0%B5%D0%B4%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5-youtube-%D0%B2-%D1%80%D0%BE%D1%81%D1%81%D0%B8%D0%B8/8074/)

https://joyreactor.cc/post/5884959

https://github.com/ValdikSS/GoodbyeDPI/issues/378

# Marzban

https://habr.com/ru/articles/771892/

Рекомендуемые клиенты для устройств:

iOS:  
[Streisand](https://apps.apple.com/app/id6450534064) (iOS 14+)  
[Shadowrocket](https://apps.apple.com/us/app/shadowrocket/id932747118) (iOS 11+)  
[FoXray](https://apps.apple.com/us/app/foxray/id6448898396) (iOS 16+)

Android:  
[v2rayNG](https://github.com/2dust/v2rayNG/releases)  
[Hiddify-Next](https://play.google.com/store/apps/details?id=app.hiddify.com)  
[NekoBox](https://github.com/MatsuriDayo/NekoBoxForAndroid/releases)

Windows:  
[Hiddify-Next](https://github.com/hiddify/hiddify-next/releases/)  
[NekoRay](https://github.com/MatsuriDayo/nekoray/releases)  
[v2rayN](https://v2rayn/)

macOS:  
[Hiddify-Next](https://github.com/hiddify/hiddify-next/releases/latest/download/hiddify-macos-universal.zip)  
[Foxray](https://apps.apple.com/us/app/foxray/id6448898396)  
[V2Box](https://apps.apple.com/us/app/v2box-v2ray-client/id6446814690)