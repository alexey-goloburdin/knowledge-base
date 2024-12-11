# Questions

# Вопросы

- Как поставить тире в любой программе?
- Как поставить кавычки ёлочкой?
- Как переключаться по вкладкам используя ALT вместо CTRL?
- Переключение языка по ALT+Space?

# Обновление Windows, драйверов

В первую очередь обновляем на новом ноутбуке саму систему, драйвера.

# Windows-софт

- Audacity
- [SumatraPDF](https://www.sumatrapdfreader.org/free-pdf-reader). Интерфейс не перегружен (его почти нет) и запускается быстро. Даже полосу прокрутки можно скрыть. Возможна навигация через jk
- Obsidian
- Davinci Resolve
- vlc as player
- WSL (set to wsl 2):
```bash
wsl --set-default-version 2
```

# Git для Windows для Obsidian

Install [Git for Windows](https://git-scm.com/downloads/win) for Obsidian. Set up it in Git bash (`git config ...`).

# Синхронизация заметок Obsidian через git

Git в Windows настроен. Плагин git в Obs и хоткей на CTRL+P для push.

# Linux

```bash
sudo apt install -y \
    zsh git gpg pass zip unzip \
    curl wget tmux gcc bsdmainutils htop ripgrep fzf bat build-essential \
    ripgrep

sudo ln -s $(which batcat) /usr/local/bin/bat

# GPG and SSH keys (change your username)
mkdir /mnt/c/Users/sterx/AppData/Roaming/alacritty/
vim /mnt/c/Users/sterx/AppData/Roaming/alacritty/alacritty.toml
```

Insert:

```toml
[general]
import = [
    "C:\\Users\\sterx\\AppData\\Roaming\\alacritty\\themes\\themes\\rose_pine.toml"
]

[env]
TERM = "xterm-256color"

[font]
size = 12

[font.normal]
family = "Hack Nerd Font"
style = "Regular"

[[keyboard.bindings]]
action = "SpawnNewInstance"
key = "N"
mods = "Command"

[[keyboard.bindings]]
chars = "\u0015"
key = "Back"
mods = "Super"

[[keyboard.bindings]]
chars = "\u001Bb"
key = "Left"
mods = "Alt"

[[keyboard.bindings]]
chars = "\u001Bf"
key = "Right"
mods = "Alt"

[[keyboard.bindings]]
chars = "\u0000"
key = "Space"
mods = "Control"

[[keyboard.bindings]]
chars = "\u001BOH"
key = "Left"
mode = "AppCursor"
mods = "Command"

[[keyboard.bindings]]
chars = "\u001BOF"
key = "Right"
mode = "AppCursor"
mods = "Command"

[[keyboard.bindings]]
chars = "\u001BR"
key = "R"
mods = "Command"

[window]
decorations = "None"
dynamic_padding = true
opacity = 1
startup_mode = "Maximized"

[window.padding]
x = 15
y = 15

[terminal.shell]
program = "wsl"
args = [
    "-d",
    "Debian",
    "--cd",
    "~"
]
```

Config Linux:

```bash
# Install oh-my-zsh
# https://ohmyz.sh/

# Download nvim into ~/.soft
# https://github.com/neovim/neovim/releases
wget https://github.com/neovim/neovim/releases/download/v0.10.2/nvim-linux64.tar.gz
tar -xzvf nvim-linux64.tar.gz
mv nvim-linux64 .soft/nvim
sudo ln -s $HOME/.soft/nvim/bin/nvim /usr/local/bin/nvim
nvim

echo "alias n=nvim" >> ~/.zshrc && . ~/.zshrc
echo "export EDITOR=vim" >> ~/.zshrc && . ~/.zshrc

# INSTALL LSP SERVERS AND OTHER TOOLS

# Download rust-analyzer into ~/.soft
# https://github.com/rust-lang/rust-analyzer/releases
cd ~/.soft
gunzip rust-analyzer-x86_64-unknown-linux-gnu.gz
sudo ln -s $HOME/.soft/rust-analyzer /usr/local/bin

# Install nodejs and pyright
# https://nodejs.org/en/download/package-manager
# choose linux and nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
export NVM_DIR="$HOME/.nvm"
nvm install 23
node -v
npm -v

npm i -g npm
npm i -g pyright

echo "def main(name: str) -> None: print(f'hello, {name}!')" > main.py
pyright main.py

# Install typescript LSP server
npm install -g typescript-language-server typescript

# Set up git in WSL
git config --global alias.st status
git config --global user.name "Alexey Goloburdin"
git config --global user.email "sterx@rl6.ru"
# чтобы кириллические имена файлов нормально выводились
git config --global core.quotepath

# Packer for nvim
mkdir -p ~/.local/share/nvim/site/pack/packer/start/

git clone --depth 1 https://github.com/wbthomason/packer.nvim \
    ~/.local/share/nvim/site/pack/packer/start/packer.nvim

# Build telescope
cd ~/.local/share/nvim/site/pack/packer/start/telescope-fzf-native.nvim
make

cd
```

# Tmux
 
tmux prefix это `CTRL`+`A` или `CTRL`+`B`, а CTRL это Caps lock. Покажи создание панелей в tmux.

```bash
git clone https://github.com/gpakosz/.tmux.git /tmp/tmux
mkdir -p ~/.config/tmux
cp /tmp/tmux/.tmux.conf ~/.config/tmux/tmux.conf
cp /tmp/tmux/.tmux.conf.local ~/.config/tmux/tmux.conf.local

# append to ~/.config/tmux/tmux.conf.local
# Custom settings
tmux_conf_theme_status_left=" ❐ #S "
tmux_conf_theme_status_right=" #{prefix} #{?battery_percentage, #{battery_percentage},} , %d %b %R "

tmux_conf_theme_status_left_fg="$tmux_conf_theme_colour_6"
tmux_conf_theme_status_left_bg="$tmux_conf_theme_colour_3"

tmux_conf_theme_status_right=""

# Использовать vi-режим в copy-mode

set -g mode-keys vi

# Включить копирование в системный буфер (требуется `xclip` или `pbcopy`)

bind-key -T copy-mode-vi y send -X copy-pipe-and-cancel "xclip -selection clipboard -i"

# Альтернативный вариант для macOS
# bind-key -T copy-mode-vi y send -X copy-pipe-and-cancel "pbcopy"

# Удобный способ входа в copy-mode
bind-key [ copy-mode

# Настройка кнопок для навигации (опционально)
bind-key -T copy-mode-vi v send -X begin-selection
bind-key -T copy-mode-vi y send -X copy-selection-and-cancel
```

# Копирование из WSL в буфер Windows

Копировать текст из WSL можно очень удобно с `clip.exe`. На маке есть `pbcopy`, а тут вот `clip.exe`:

```bash
ls -la /mnt/c/Users/
history | tail -1 | clip.exe
```

Можно использовать для копирования паролей из `pass`, например. Можно перебиндить на pbcopy при желании:

```bash
echo "alias pbcopy=clip.exe" >> ~/.zshrc && . ~/.zshrc
```

Открытие проводника в текущей директории (удобно, использовал `open .` на маке):

```bash
echo "alias open=explorer.exe" >> ~/.zshrc && . ~/.zshrc
```

# Настройки ввода

Уменьшим задержку при вводе повторных символов. **Win + R**, команда `control keyboard`, **Enter**. Откроется окно "Свойства клавиатуры". В разделе **Задержка перед началом повторения** (Repeat delay) установите ползунок ближе к значению **Короткая** (Short).

# Горячие клавиши в системе

AutoHotkey -- программа для настройки комбинаций клавиш.

Set up hot keys:

```bash
nvim "/mnt/c/Users/sterx/AppData/Roaming/Microsoft/Windows/Start Menu/Programs/Startup/hotkeys.ahk"

CapsLock::Ctrl
```
 
Then reboot.

Доступ к терминалу всегда по Win+1. Очень удобно. К Obsidian -- по Win+2. Chrome -- Win+3. И так далее  для всего нужного софта. У меня пока только эти три программы. TG запускаю через Win и поиск.

Поиск на маке через spotlight был по CTRL+Пробел у меня настроен, тут просто по кнопке Win. И запуск софта, и поиск файлов (покажи поиск по слову Сценарий).

# Отключение всех системных звуков

Отключаю все системные звуки в системе, чтобы не мешали, раздражают.

---

После 15 лет работы на маках, после отличного эйра на М1 купить виндоус-ноут это настоооолько бредовая идея, что... она мне показалась даже интересной! И я купил себе 
windows-ноутбук. Да. Ну а чо нет.

И я не просто его купил. Я его протестил, настроил, и последнюю неделю полноценно работаю только на нём.

Расскажи в тч про волпаперы:
- https://goodfon.ru
- https://wallhaven.cc/toplist
- https://pexels.com
- https://unsplash.com

Про Huawei. Не было денег, купил телефон. Время было тяжелое - можно было вернуться в найм на отличную ЗП, потому что в бизнес я ушёл из отличных компаний, Oracle и SAP, c очень хорошими зарплатами, бонусами, возможностями и тд. И можно было плюнуть на свои проекты, которыми хотелось заниматься, отказаться от них, признать свою неудачу, и вернуться в тот же Oracle. В общем было тяжелое время, ничего не получалось, денег не было и я был очень рад возможности купить себе отличный телефон Huawei.

Я работаю на маке года с 2009го. Ещё в студенческое время, день посвящая учёбе, а ночь айтишке и фрилансу, я заработал на свой первый MacBook Pro 13 и с тех пор кайфую, работая на маках.

Около полу года у меня был опыт работы на БУ-шном ThinkPad с Авито, который я купил, продав свой мак, потому что нужны были деньги, накатил на этот ThinkPad Linux и благополучно страдал, потому что WiFi на линуксе рандомно отваливался раз в десять или 20 или 30 минут. Была ошибка в драйвере и никто даже не собирался её чинить, потому что  пошёл ты нахрен, чёт не нравится, возьми и сам себе драйвер напиши, умник!

И затем я снова вернулся на мак.

Расскажи о характеристиках своего Huawei Matebook X Pro 2024

Fn+r - переключение герцовки экрана, быстро-удобно.
Fn+p - performance более производительный или более долгоживущий режим.

Сравни отражения на выключенном экране света сзади с Air и Pro маками

Запись скринкаста постукиванием по тачпаду костяшками

Тачпад на экране -- юзабельно, тыкаю иногда, скроллю иногда как по планшету, читая что-то

По тачпаду можно водить для изменения яркости и громкости. Я этим не пользуюсь, мне нравится управлять этим с Fn-клавишами, но кому-то будет удобно возможно. Фейковых нажатий нет, что важно, то есть нет такого, что вместо сдвига мыши через тачпад меняется звук или яркость экрана, такого ни разу не было.

В левый верхний экран -- свернуть это приложение. Правый верхний угол -- закрыть окно. Поиграться прикольно, но я отключил это поведение -- мешает. Управляю окнами с клавиатуры. CTRL+F4 закрывает окно, причем даже когда ряд F-кнопок работает как медиа-клавиши. То есть F4 отключает звук, а CTRL+F4 закрывает окно. Не знаю, это предусмотрели в Huawei или в Windows, вангую, что в Huawei -- молодцы.

Свернуть все окна Win+D или Win+M.

Заряжать с любой стороны, type c, удобно, компактная зарядка и не обязательно родная. На многих виндоус ноутах да и на мощных макбуках зарядка там уже ощутимая и по размеру и по весу. Здесь в целом можно от 65вт зарядки заряжать, и от чуть более мощной 90 ваттной.

Магниевый корпус - легче чем алюминий и есть мнение, что крепче. Покрытие офигенно тактильно приятное и не стирается как на макбуках (мой потерся под запястьями, все крашеные стираются). Здесь как-то сделано, что по отзывам не стирается, металл царапается, а тут гораздо менее нежный материал, гораздо менее нежный. И, повторюсь, тактильно офигенно. Разные цвета немного разные тактильно, пробуйте, черный мне тактильно не понравился и он ляпается очень сильно, а этот хорош: белый хорош, розовый хорош. Оооочень приятный тактильно.

Забор воздуха сбоку, а не снизу -- можно использовать на кровати или на коленях. На Honor снизу и на многих других снизу, на асусах я вот смотрел обзор снизу. Тут сбоку и это хорошо.

Яркий экран -- сравни с Air и Pro

Офигенный звук, действительно офигенный. 6 динамиков.

Прямой размыкатель для камеры, это не софтверная штука, а именно хардверная, размыкается цепь к камере. Те кто разбирали ноутбук, это подтверждают.

Davinci Resolve compare

