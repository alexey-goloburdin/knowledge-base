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

# Работа с версиями Python

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

# Запуск Python-кода

```bash
uv run main.py

uv add requests
# без инициализации проекта можно, прям так
bat pyproject.toml
# этот файл — это альтернатива pip freeze, современная и удобная
# pip тоже понимает этот формат файла, если что, то есть установить пакеты,
# описанные в этом файле, он может

nvim main.py
```

```python
from pprint import pprint

import requests

countries = requests.get("https://restcountries.com/v3.1/name/russia").json()
pprint(countries)
```

Или можно с инициализацией проекта, чтобы добавить туда `git`, `.gitgnore` и тд:

```bash
uv init myproject
```

Или в текущем проекте в его директории можно выполнить `uv init`.

Можно вот так красиво показать дерево зависимостей, а, кайф?

```bash
uv tree
```

Тут не надо создавать самому вирт окружение. Можно, но не надо:

```bash
uv add django
uv run django-admin startproject myproject
ls
cd myproject
ls
uv run manage.py startapp firstapp
uv run manage.py runserver
```

На самом деле вирт окружение создастся в директории `.venv`.

Также можно для отдельного скрипта подтянуть любую библиотеку — вот вообще без создания вирт окружения:

```bash
uv run --with httpx==0.26.0 python -c "import httpx; print(httpx.get('https://to.digital').text[:151])"
```

Скажите, класс, а?

# uvx

```bash
uvx ruff
# download it

# from directory with manage.py
uvx ruff check .

uvx ruff check . --fix
uvx ruff check .
```

Это работает в своём изолированном окружении. Если надо в окружении проекта, то можно по классике затащить зависимость в проект и запускать её с `uv run`:

```bash
# from directory with manage.py
uv add --dev pytest

ls
# обрати внимание — pytest добавился в pyproject.toml на уровне выше
bat ../pyproject.toml

uv run pytest
```

# pyright

`pyrightconfig.json`:

```json
{
    "venvPath": ".",
    "venv": ".venv"
}
```

# Совместимость с pip

Также в uv есть режим совместимости с `pip`, например, если у нас есть файл зависимостей, полученный в результате выполнения команды `pip freeze`, то мы можем установить эти зависимости таким образом:

```bash
uv pip install -r requirements.txt
```

# Ещё

Помимо `uv` они сделали ещё линтер `ruff`, тоже быстрый, тоже кайфовый, сделаю о нём отдельное видео.

А сейчас эти ребята делают ещё свой статический анализатор типов как mypy и pyright. Ждём!
