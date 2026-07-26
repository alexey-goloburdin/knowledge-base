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


