# Questions

- Как поставить тире в любой программе?
- Как поставить кавычки ёлочкой?
- Как переключаться по вкладкам используя ALT вместо CTRL?
- Переключение языка по ALT+Space?

AutoHotkey -- программа для настройки комбинаций клавиш.

Davinci Resolve как работает?

Скорость компиляции Python 3.12 сравни?

Покажи настройку WSL -- терминала, Alacritty, установленного софта:
- ssh and gpg keys
- pass

Syncthing, Obsiidian
- как происходит выгрузка в git напрямую из Obsidian?


Windows soft:
- audacity
- [SumatraPDF](https://www.sumatrapdfreader.org/free-pdf-reader). Интерфейс не перегружен (его почти нет) и запускается быстро. Даже полосу прокрутки можно скрыть. Возможна навигация через jk
- 

Размещение софта на панели. Первое место Alacritty, всегда в максимально быстром доступе линукс-консоль, которую я не закрываю. Можете поставить сюда Windows Terminal, если хотите, или другой терминал - Kitty, WezTerm и тд.


```bash
sudo apt install -y \
    zsh git gpg pass zip unzip \
    curl wget tmux gcc htop ripgrep fzf bat build-essential

sudo ln -s $(which batcat) /usr/local/bin/bat

# Install oh-my-zsh
# https://ohmyz.sh/

# INSTALL LSP SERVERS AND OTHER TOOLS

# Download rust-analyzer into ~/.soft
# https://github.com/rust-lang/rust-analyzer/releases
cd ~/.soft
gunzip gunzip rust-analyzer-x86_64-unknown-linux-gnu.gz
sudo ln -s $HOME/.soft/rust-analyzer /usr/local/bin

# Download nvim into ~/.soft
# https://github.com/neovim/neovim/releases
wget https://github.com/neovim/neovim/releases/download/v0.10.2/nvim-linux64.tar.gz
tar -xzvf nvim-linux64.tar.gz
mv nvim-linux64 .soft/nvim
sudo ln -s $HOME/.soft/nvim/bin/nvim /usr/local/bin/nvim
nvim

echo "alias n=nvim" >> ~/.zshrc && . ~/.zshrc

# Install ripgrep
sudo apt install -y ripgrep

# Install nodejs
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


# Set up git
git config --global alias.st status
git config --global user.name "Alexey Goloburdin"
git config --global user.email "sterx@rl6.ru"
# чтобы кириллические имена файлов нормально выводились
git config --global core.quotepath

# packer
mkdir -p ~/.local/share/nvim/site/pack/packer/start/

git clone --depth 1 https://github.com/wbthomason/packer.nvim \
    ~/.local/share/nvim/site/pack/packer/start/packer.nvim

# Build telescope
cd ~/.local/share/nvim/site/pack/packer/start/telescope-fzf-native.nvim
make

cd



```

Install [Git for Windows](https://git-scm.com/downloads/win) for Obsidian. Set up it in Git bash (`git config ...`).

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

Доступ к терминалу всегда по Win+1. Очень удобно. К Obsidian -- по Win+2. Chrome -- Win+3. И так далее.

Поиск на маке через spotlight был по CTRL+Пробел у меня настроен, тут просто по кнопке Win. И запуск софта, и поиск файлов (покажи поиск по слову Сценарий).

Отключаю все системные звуки в системе, чтобы не мешали, раздражают.

---

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

