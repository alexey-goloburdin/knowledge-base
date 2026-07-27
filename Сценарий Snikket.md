# Упомяни курс

Здоров, котаны! 

Сегодня мы с вами установим и настроим и протестируем свой мессенджер с аудио и видео звонками, по сути аналог WhatsApp, Telegram, Max, что там ещё есть.

Чтобы всё было установлено на свой сервер, чтобы всё шифровалось, чтобы всё работало вне зависимости от VPN, включен он у вас или выключен, чтобы использовался строго open source софт, то есть программы с открытым исходным кодом, чтобы везде работали нормально уведомления, на Android и айфонах, чтобы можно было пользоваться также и на Windows и на маке и на Linux.

Покажу, как работает та система, которую мы сегодня настроим.

==показываю чат и звонок==

Как всегда, есть таймкоды, поэтому можете быстро перейти к интересующему вас разделу. Начну с короткой предыстории, зачем мы всем этим занимаемся.

Как вы знаете, звонки в Telegram в России больше не алё, что с VPN, что без него. Звонки через ВК и Max больше тоже не алё, потому что клиенты этих звонилок удалены из App Store и потому на айфонах не работают уведомления о сообщениях и звонках. То есть вы звоните, например, родителям, а у них нет никакого уведомления о входящем звонке. С этим можно пытаться как-то бороться, выносить веб-версию на рабочий стол, но это всё извращение и всё равно работает криво-косо, не надёжно, не нативно и так далее.

Здесь стоит сказать, что конечно, нам нужен не аналог системы телеконференции, вроде Zoom, Google Meet, Яндекс.Телемоста и прочего. Нам нужен именно мессенджер с аудио и видеозвонками. Что-то похожее, например, на Telegram или WhatsApp, чтобы, скажем, созваниваться с родными, хотя то, что мы сейчас настроим, можно использовать и для коммуникации в команде, там и групповые чаты есть, и все, что нужно. Но разные системы, заточенные на конференции, вроде Jitsi, Nextcloud Talk и подобных, здесь не подходит.

Итак, ставить мы будем Snikket. Это готовый полностью автоматизированный XMPP-сервер с удобной админкой для управления. Настроили и всё просто работает. А клиентских программ, через которое пользователи будут чатиться и созваниваться — огромное количество, потому что протокол XMPP стандартизирован. Я им пользовался в универе еще году в 2006м, тогда его называли Jabber.

Итак, будем ставить Snikket на свой сервер.  Сразу скажу, что просто прочесть документацию Snikket и сделать всё по ней, к сожалению, недостаточно — мой опыт показал, что и дефолтные клиенты Snikket отстой, и дополнительные настройки требуются для корректной работы видеозвонков, поэтому всё это покажу и вам не придется собирать эти грабли самостоятельно.

Что нам понадобится. Нам понадобится свой сервер, который мы сейчас арендуем в Selectel, я полностью покажу этот процесс, и нам понадобится домен, который вы также можете купить в Selectel, я тоже покажу, как это сделать. Конечно, процесс настройки сложнее, чем просто установить готовый мессенджер аля WhatsApp, надо будет прямо в терминале вводить всякие разные команды, яНо я вам полностью этот процесс покажу.




https://snikket.org/service/quickstart/

```shell
adduser www
usermod -aG sudo www
su - www
sudo apt update
sudo apt install -y zsh git
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# local computer
cat ~/.ssh/id_ed25519.pub | pbcopy

# remote computer with root auth
install -d -m 700 -o www -g www /home/www/.ssh
vim /home/www/.ssh/authorized_keys
# paste public key

chown www:www /home/www/.ssh/authorized_keys
chmod 600 /home/www/.ssh/authorized_keys
sshd -t && systemctl reload ssh

# test from local from parallel terminal
ssh www@155.212.180.31
# закроем вход для рута
sudo tee /etc/ssh/sshd_config.d/00-disable-root.conf >/dev/null <<'EOF'
PermitRootLogin no
EOF

sudo sshd -t && sudo systemctl reload ssh

# install docker
# https://docs.docker.com/engine/install/ubuntu/
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo systemctl status docker
sudo docker run hello-world

df -h
```

https://my.selectel.ru/network/domains
to.digital domain

```
sn.to.digital  300  IN     A     155.212.180.31
groups.sn.to.digital  300  IN     CNAME  sn.to.digital
share.sn.to.digital   300  IN     CNAME  sn.to.digital
```

server:

```shell
mkdir /etc/snikket
cd /etc/snikket
sudo curl -o docker-compose.yml https://snikket.org/service/resources/docker-compose.yml

sudo tee snikket.conf <<EOF
# The primary domain of your Snikket instance
SNIKKET_DOMAIN=sn.to.digital

# An email address where the admin can be contacted
# (also used to register your Let's Encrypt account to obtain certificates)
SNIKKET_ADMIN_EMAIL=sterx@rl6.ru

SNIKKET_TLS_PROFILE=intermediate
EOF

# printf '\nSNIKKET_TLS_PROFILE=intermediate\n' | sudo tee -a snikket.conf
sudo docker compose up -d
sudo docker exec snikket create-invite --admin --group default
```


- iOS и MacOS — Monal, https://monal-im.org/install/
- Android — Conversations из F-Droid
- Windows — https://dinox.im/download/
- 