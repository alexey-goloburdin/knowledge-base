https://wiki.aeza.net/razvertyvanie-proksi-protokola-vless-s-pomoshyu-marzban#id-2.3-nastroika-protokola-vless-tcp-reality-v-paneli-marzban-i-sozdanie-polzovatelya

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

install:

```bash
sudo bash -c "$(curl -sL https://github.com/Gozargah/Marzban-scripts/raw/master/marzban.sh)" @ install
```

Когда установка будет завершена:

- Вы увидите логи, которые можно остановить, нажав `Ctrl+C` или закрыв терминал.
- Файлы Marzban будут размещены по адресу `/opt/marzban`.
- Файл конфигурации будет размещен по адресу `/opt/marzban/.env`
- Файлы с данными будут размещены по адресу `/var/lib/marzban`.
- Вы можете получить доступ к панели управления, введя в адресной строке `http://YOUR_SERVER_IP:8000/dashboard/` (заменив YOUR_SERVER_IP на актуальный IP адрес вашего сервера).

Далее, Вам нужно создать главного администратора для входа в панель управления Marzban, выполнив следующую команду:

```bash
marzban cli admin create --sudo
```

На этом этапе уже можно пользоваться панелью администратора, но для работы подписок, нод и безопасного доступа по https нужно произвести следующие действия:

Файлы сертификатов должны быть доступны по адресу `/var/lib/marzban/certs`, чтобы Marzban мог получить к ним доступ.

Прежде чем приступить к получению SSL-сертификата, вы должны настроить DNS-записи домена.

Получаем сертификат с acme.sh:

Для начала необходимо установить socat и cron (cron обычно уже установлен, поэтому команда проверит наличие)

```bash
apt install cron socat
```


Устанавливаем acme.sh

EMAIL = Ваш email

```bash
curl https://get.acme.sh | sh -s email=EMAIL
```

Создаем директорию для сертификатов

```bash
mkdir -p /var/lib/marzban/certs/
```

Получаем сертификат

Введите ваш домен или субдомен в поле `DOMAIN`

```bash
~/.acme.sh/acme.sh --set-default-ca --server letsencrypt  --issue --standalone -d DOMAIN \--key-file /var/lib/marzban/certs/key.pem \--fullchain-file /var/lib/marzban/certs/fullchain.pem
```

_В случае, если вам необходимо получить сертификаты для ваших поддоменов, команда получения сертификата будет выглядеть так_

```bash
~/.acme.sh/acme.sh --set-default-ca --server letsencrypt --issue --standalone \-d DOMAIN \-d SUBDOMAIN1.DOMAIN \-d SUBDOMAIN2.DOMAIN \--key-file /var/lib/marzban/certs/key.pem \--fullchain-file /var/lib/marzban/certs/fullchain.pem
```

Проверить список выпущенных сертификатов:

```
~/.acme.sh/acme.sh --list
```

Далее добавляем сертификаты SSL в Marzban

При включении SSL в Marzban, панель управления и ссылка на подписку будут доступны через https.

Во всех примерах ниже вы можете найти файлы `docker-compose.yml`и `.env`по пути `/opt/marzban‍‍‍`, а `xray_config.json`по пути `/var/lib/marzban`

Marzban запускается по умолчанию с помощью`Uvicorn` , он же позволяет вам определять файлы сертификатов SSL.

После создания файлов сертификатов SSL установите в файле `.env` следующие переменные .

`YOUR_DOMAIN` - ваш домен или субдомен

```bash
vim /opt/marzban/.env
```

Изменяем в нем следующие переменные

```bash
UVICORN_PORT = 443
UVICORN_SSL_CERTFILE = "/var/lib/marzban/certs/fullchain.pem"
UVICORN_SSL_KEYFILE = "/var/lib/marzban/certs/key.pem"
XRAY_SUBSCRIPTION_URL_PREFIX = https://YOUR_DOMAIN
```

Теперь панель управления Marzban будет доступна на вашем домене или субдомене по https. Вы можете получить доступ к панели управления, введя в адресной строке `https://YOUR_DOMAIN/dashboard/`

**Поскольку master версия имеет xray 1.8.1 (актуальный 1.8.4) не имеет русского языка и рабочего telegram бота, лучше сразу перейти на версию dev**

Переход на версию для разработчиков (dev):

```bash
cd /opt/marzban
```

```bash
vim docker-compose.yml
```

Измените третью строку с `marzban:latest` на `marzban:dev` и сохраните изменения

выполните обновление

```bash
marzban update 
```

Для добавления Telegram бота:

```bash
vim /opt/marzban/.env
```

Убираем # перед TELEGRAM_API_TOKEN и TELEGRAM_ADMIN_ID и вносим данные, для Telegram_API_TOKEN берем данные вашего бота из Tg@BotFather, для TELEGRAM_ADMIN_ID из TG@userinfobot или иного источника, где можно узнать ID пользователя-администратора

### Настройка клиентской конфигурации

Переходим в панель администратора. Далее нужно создать новую конфигурацию пользователя. Можно задать имя пользователя, лимит трафика и срок сброса лимита, дата истечения конфигурации и примечание. Можно добавить как все конфигурации разом, так и выбрать только нужные вам, **для XTLS-Reality оставляем только Vless**.

Для копирования ссылки на подписку на конфигурации - первая кнопка, для копирования созданных конфигураций - вторая, для просмотра QR-кодов подписки и конфигураций - третья кнопка. Детальные настройки Inbounds описаны в [Wiki проекта](https://docs.marzban.ru/start/host-settings/).


