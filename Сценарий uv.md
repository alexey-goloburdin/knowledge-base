Здоров, котаны, если вдруг вы упустили современный Python-инструментарий, то рад познакомить вас с `uv`! Это пакетный менеджер для Python, как pip или poetry, только лучше, а ещё как pyenv, только лучше, а ещё как virtualenv, только лучше. Вот прям всё в одном. И вот прям blazingly fast, вот прям на расте, прям пушечка!

Давайте разбираться!

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

- `uv python list` — посмотреть список установленных на компьютере версий python. Видит в том числе установленные в системе версии 
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
# Запуск скриптов без виртуального окружения

Также можно для отдельного скрипта подтянуть любую библиотеку — вот вообще без создания вирт окружения:

```bash
uv run --with httpx==0.26.0 python -c "import httpx; print(httpx.get('https://to.digital').text[:151])"
```

Скажите, класс, а?

Но на этом ещё не заканчивается. Можно прям в файле прописать, что нам нужно, все зависимости и версию интерпретатора. И запускать этот файл!

У меня есть скрипт, который нумерует маркдаун-файлы в директории. Я написал его для себя, я пишу курс Хардкорная веб-разработка в markdown-документах в Obsidian, и все уроки нумеруются, и иногда мне надо вставить урок в середину и не хочется менять нумерацию всех последующих уроков. Тогда я просто запускаю свой пайтон-скрипт и он делает красиво. Вот так выглядит этот код:

```python
import argparse
from pathlib import Path
import os
import re

from natsort import natsorted
from tabulate import tabulate

parser = argparse.ArgumentParser()
parser.add_argument('-s', '--simulate', action='store_true', help="simulate renaming")
parser.add_argument('base', nargs='?', default="", help="base static number — before counter")
args = parser.parse_args()


for root, dirs, files in Path(".").walk():
    files = tuple(filter(lambda f: f.lower().endswith(".md"), natsorted(files)))
    if not files: continue
    index = 0

    max_filename_len = len(sorted(files, key=lambda f: len(f))[-1])

    files_for_rename, renamed_files = [], []
    for file in files:
        match = re.match(r"^([\d\. ]*)(.*)", file)
        if not match: raise Exception(f"strange {file=}")
        match = match.groups()
        file_name = match[1]

        index += 1
        base_number = "" if not args.base else f"{args.base}."
        new_file_name = f"{base_number}{index}. {file_name}"
        files_for_rename.append([file, new_file_name])

    try:
        for filenames in files_for_rename:
            file, new_file_name = filenames
            if not args.simulate:
                os.rename(file, new_file_name)
            renamed_files.append(filenames)
    except Exception as e:
        print(e)

    print(tabulate(renamed_files, headers=["old name", "new name"]))

if args.simulate:
    print("Changes simulated")
```

Покажу, как работает:

```bash
cd files
ls

true > "2.2. Как всё устроено — кластер, база данных, схема, табличное пространство, файл, страница.md"

uv run --with natsort --with tabulate ../number_files.py --simulate
uv run --with natsort --with tabulate ../number_files.py --simulate 12
```

Здесь используется 2 внешних либы — natsort и tabulate. Надо создавать виртуальное окружение ради одного этого скрипта и это, честно говоря, уныло. `uv` поможет! Можно запустить это так:

```bash
uv run --with natsort --with tabulate number_files.py
```

И без виртуального окружения! Кайф!

```text
#!/usr/bin/env -S uv run --script

# /// script
# requires-python = ">=3.13"
# dependencies = [
#   "natsort",
#   "tabulate"
# ]
# ///
```

# Точки входа

Также можно создавать команды или точки входа, добавим в `pyproject.toml`:

```
[project.scripts]
hello = "main:hello"
```

Запустим:

```bash
echo "def hello(): print('hello!')" > main.py
uv run helllo
```

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
