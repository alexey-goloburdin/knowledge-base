# Установка

```bash
# for Mac OS or Linux
curl -LsSf https://astral.sh/uv/install.sh | sh
# если вдруг нет curl на чистой инсталляции linux — сначала установите его:
sudo apt install -y curl

# for Mac OS with brew
brew install uv

# autocomplete for bash or zsh
echo 'eval "$(uv generate-shell-completion bash)"' >> ~/.bashrc
echo 'eval "$(uv generate-shell-completion zsh)"' >> ~/.zshrc

echo 'eval "$(uvx --generate-shell-completion bash)"' >> ~/.bashrc
echo 'eval "$(uvx --generate-shell-completion zsh)"' >> ~/.zshrc
```

Основные команды:

- `uv list` — посмотреть список установленных на компьютере версий python. Видит в том числе установленные в системе версии 
- `uv python install 3.13.2` — поставить конкретную версию. У разработчиков `uv` есть свои собранные бинари и они качаются и ставятся, то есть это не сборка питона с нуля. Насколько это будет работать в разных версиях дистрибутивов linux — не знаю, не тестил, актуальные версии python на актуальных версиях debian таким образом ставятся нормально
- `uv python pin 3.13.2` — создаст `.python-version` с нужной версией python. Для `uv run` и для других команд будет использовать эту версию Python

```python
# main.py
import sys

print("hello", sys.version)
```

Запускаем:

```bash
uv run main.py

uv python pin 3.13.2
uv run main.py

```

Круто, да? Просто создали текстовый файл с версией питона и остальное сделаем uv:

```bash
uv python uninstall 3.13.2
bat .python-version

uv run main.py
# download and run

uv python list
```