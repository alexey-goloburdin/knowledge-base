>[!info] Добавь рекламу курса.

Ооо, здоров, котаны, огненный материал сегодня для вас подготовил! Сам бы с кайфом хотел такой посмотреть:)

Напишем сегодня свой агент для разработки, такой свой минимальный claude code на 99 строк кода, а его основное ядро, отвечающее за основную логику, займет всего 6 строк. Вообще серьёзно.

Поговорим про agent loop и про то, как под капотом работают все более-менее автономные ИИ-агенты, например, Claude Code, Codex, OpenCode, pi, OpenClaw, Hermes и прочие (==покажи их названия==). Это очень интересно с одной стороны, и удивительно легко и просто с другой стороны. Дело в том, что ключевая функциональность всех этих агентов, вот ядро, может быть реализована в 6 строках кода. Вот эти 6 строк на питоне:

```python
while True:
    response = callLLM(messages)
    if response.toolCalls:
        messages.append(executeTools(response.toolCalls))
    else:
        return response
```

Мы дальше подробнее разберём этот код и напишем на его основе своего автономного кодинг ассистента. Ну и если вы хотите разрабатывать своих автономных агентов, необязательно для написания кода, и не понимаете, как они работают, то этот материал ровно чётенько для вас. Потому что цель у агента может быть любой, это небязательно написание кода, а принципы работы агента будут ровно те же самые.

Короче. Что такое ИИ-агент? Определений много, все они подчёркивают разные особенности и смотрят на агентов с разных сторон, но давайте назовём по-простому ИИ-агентом связку большой языковой модели и доступных ей инструментов. То есть агент вызывает большую языковую модель через API и передаёт ей помимо промпта ещё набор инструментов, то есть функций с их названиями и описаниями параметров. И модель в ответе может просить агента вызвать эти инструменты с такими-то аргументами. Агент вызывает инструменты и передаёт результат их вызова обратно модели. И в ответ модель может или вернуть конечный ответ пользователю или снова вернуть просьбу вызвать какой-то инструмент и так далее, пока цель не будет достигнута и задача не будет решена.

Вот это и есть тот самый agent loop (==покажи agent loop==), агентский цикл, код которого я уже показывал:

```python
while True:
    response = callLLM(messages)
    if response.toolCalls:
        messages.append(executeTools(response.toolCalls))
    else:
        return response
```

У нас тут есть бесконечный цикл `while True`. Структура `messages` это список всех сообщений в модель и от модели, то есть история контекста. Функция `callLLM` отвечает за вызов модели с переданным ей контекстом. Эта функция возвращает ответ от модели, в этом ответе может быть просьба вызвать инструмент или может быть просто ответ. Если есть просьба вызвать инструмент, то этот инструмент вызывается в функции `executeTools` и результат от инструмента тоже добавляется в контекст. И цикл уходит на следующую итерацию. Выход из цикла происходит тогда, когда модель прекратила возвращать просьбу вызвать инструмент, это считается либо решённой задачей, либо ситуацией, когда необходимо подключиться человеку, то есть нужно какое-то действие или уточнение от человека.

На практике этот цикл, конечно, необязательно будет бесконечным, и часто имеет смысл установить какое-то разумное ограничение на максимальное количество итераций.

Иии — вот и всё. По сути так и работают все автономные ИИ-агенты. Конечно, вокруг этого накручивается дополнительная логика на проверки безопасности, на сжатие контекста, когда он приближается к ограничению контекстного окна, на устранение зацикливания, когда модель начинает вызывать одни и те же инструменты с одними и теми же аргументами, и так далее, но суть, ядро — ровно такая, вот эти 6 строк кода, которые в том или ином виде представлены во всех этих ИИ-агентах.

Ну что, давайте бахнем минимального кодинг ассистента. На питончике. Который будет коммуницировать с локальной LLM-моделью и шарашить нам проекты. Но для начала нам надо подумать, как наш агент будет коммуницировать с внешним миром? Например, читать файлы, записывать файлы, искать файлы, запускать проекты для их проверки? А очень просто: давайте дадим ему доступ в bash, вот ровно один инструмент. Используя bash, он сможет делать все эти и массу других великолепных операций.

Если вдруг кто не понимает, что за bash такой интересный, то я имею в виду доступ к полноценной консоли linux, unix, mac os. Ну, или консоли Windows, если вы предпочитаете Windows, но про windows мы тут не говорим. Да, безусловно, это небезопасно, давать доступ ко всей системе, но вы можете запускать агента, например, в изолированном контейнере, в изолированной виртуалке или где угодно ещё. Или накрутить поверх нашего минимального кода логику разных проверок.

Итак, а вот и наш агент, назовём его чебупелька: (`/home/sterx/code/chebupelka/chebupelka.py`):

```python
"""Minimal coding agent loop — one tool: bash."""
import json, sys, subprocess
import requests

LLM_BASE_URL = "http://192.168.2.66:1234/v1"
LLM_API_KEY = "sk-lm-8lxtW0iY:g7ImIC5vURnv2nJzxG5b"
LLM_MODEL = "qwen3.6-35b-a3b"
LLM_HEADERS = {"Content-Type": "application/json", "Authorization": f"Bearer {LLM_API_KEY}"}
MAX_TURNS = 1000


SYSTEM_PROMPT = """\
You are a coding agent. Your job is to help the user with programming tasks.

You have access to ONE tool: `bash` — which executes shell commands and returns stdout/stderr.

Workflow:
1. Plan what needs to be done.
2. Use `bash` to read files, run commands, write code, etc.
3. After gathering enough information or completing the task, give your final answer in natural language.
4. To finish, reply with a regular message (no tool call).

Be concise. Explain what you're doing before each command."""

LLM_TOOLS = [
    {"type": "function",
     "function": {"name": "bash",
                  "description": "Execute a shell command and return the output.",
                  "parameters": {"type": "object",
                                 "properties": {
                                     "command": {"type": "string", "description": "The bash command to execute."}
                                 },
                                 "required": ["command"]}
                 }
            }]


def run_bash(command: str) -> str:
    try:
        command_result = subprocess.run(command, shell=True, capture_output=True, text=True, timeout=120)
        out = command_result.stdout + (f"\nSTDERR:\n{command_result.stderr}" if command_result.stderr else "")
        return f"Exit code: {command_result.returncode}\n{out}"
    except subprocess.TimeoutExpired:
        return "Error: command timed out after 120s"


def call_tool(name: str, arguments: dict) -> str:
    func = {"bash": run_bash}.get(name)
    if not func:
        return f"Error: unknown tool '{name}'"
    try:
        return func(**arguments)
    except Exception as e:
        return f"Error calling {name}: {e}"


def call_llm(messages):
    payload = {"model": LLM_MODEL, "messages": messages, "tools": LLM_TOOLS, "tool_choice": "auto",
               "temperature": 0.1, "max_tokens": 4096}
    llm_http_response = requests.post(f"{LLM_BASE_URL}/chat/completions", json=payload, headers=LLM_HEADERS)
    llm_http_response.raise_for_status()
    msg = llm_http_response.json()["choices"][0]["message"]
    content = (msg.get("content") or "").strip()
    tool_calls = msg.get("tool_calls") or []
    return content, tool_calls


def agent_loop(user_message: str) -> None:
    messages: list[dict[str, object]] = [
        {"role": "system", "content": SYSTEM_PROMPT}, {"role": "user", "content": user_message}
    ]
    for turn in range(1, MAX_TURNS + 1):
        print(f"\n{'='*60}\n🔄 Turn {turn}\n{'='*60}")
        content, tool_calls = call_llm(messages)
        if content:
            print(f"\n🤖 {content}")
        if not tool_calls:
            print("(no text output)" if not content else "")
            print("✅ Agent finished")
            return
        prefix = "\n" if content else ""
        for tool_call in tool_calls:
            function = tool_call["function"]["name"]
            arguments = json.loads(tool_call["function"]["arguments"])
            tool_call_id = tool_call["id"]
            print(f"{prefix}🔧 Tool: {function}({json.dumps(arguments, ensure_ascii=False)})")
            result = call_tool(function, arguments)
            print(f"   → {result[:500]}{'...' if len(result)>500 else ''}")
            messages.append({"role": "assistant", "content": content or None, "tool_calls": [tool_call]})
            messages.append({"role": "tool", "tool_call_id": tool_call_id, "content": result})
    print(f"\n⚠️  Max turns ({MAX_TURNS}) reached. Stopping.")


if __name__ == "__main__":
    prompt = " ".join(sys.argv[1:]) if len(sys.argv) > 1 else ""
    if not prompt.strip():
        print("No task provided. Exiting.")
        sys.exit(1)
    agent_loop(prompt)
```

Как видим, здесь всего 99 строк кода вместе с отступами, комментами и всякой машинерией. Всего навсего 99 строк кода. Код давайте подробнее посмотрим чуть позже, а сейчас давайте посмотрим, на что способен такой минимальный агент. 

В zsh создадим для него alias для быстрого запуска:

```shell
echo 'alias chebupelka="/home/sterx/code/chebupelka/.venv/bin/python3 /home/sterx/code/chebupelka/chebupelka.py"' >> ~/.zshrc
.  ~/.zshrc
```

Теперь мы можем вызывать нашего агента, просто написав `chebupelka`.

Начнём с простого вопроса:

```python
chebupelka "чо как дела"

============================================================
🔄 Turn 1
============================================================

🤖 Привет! Всё отлично, спасибо. Я готов к работе — пиши код, решай задачи или просто болтай.

Чем могу помочь?
✅ Agent finished.
```

Здесь, как мы видим, только 1 этап коммуникации с агентом — мы спросили, он ответил, отлично.

Давайте перейдём в директорию с заданиями по 13й главе моего курса Хардкорная веб-разработка и спросим, сколько там сейчас заданий:

```python
cd $TASKS

chebupelka "это задания по одной главе моего курса, посмотри структуру вложенных директорий и скажи сколько тут всего заданий?"
```

Агент за 5 итераций агентского цикла смог ответить на вопрос. Отлично. 13я глава курса — это начальный Python, и вот в этой главе сейчас действительно 482 задания, и это число увеличивается, потому что я продолжаю их писать хахахах:) Если кто не в курсе, речь о «Хардкорной веб-разработке», моём авторском большом курсе, в котором мы глубоко копаем линукс, PostgreSQL, Python, Git, бэкенд, фронтенд, вопросы архитектуры и качества кода и так далее. Хотя многие, кто проходит курс, говорят, что самый большой его кайф в наших созвонах и чатике. Кто готов работать и хочет прокачаться — приходите. Хардкор я вам обещаю:)

Ну и давайте теперь его попробуем использовать для программирования:

```python
cd 
mkdir tmp; cd tmp

su - postgres
psql
CREATE DATABASE users_db;
CREATE USER users_db_user WITH PASSWORD 'tmp';
GRANT ALL PRIVILEGES ON DATABASE users_db TO users_db_user;
\c users_db
GRANT ALL ON SCHEMA public TO users_db_user;

CREATE DATABASE users_db_test;
CREATE USER users_db_user WITH PASSWORD 'tmp';
GRANT ALL PRIVILEGES ON DATABASE users_db_test TO users_db_user;
\c users_db_test
GRANT ALL ON SCHEMA public TO users_db_user;


bat task.md
Напиши в текущей директории на python (используй uv) fastapi-проект с CRUD пользователей, используй REST.

По каждому пользователю:
- id — bigint, autoincrement
- first_name — varchar(200), not null
- last_name — varchar(200), not null
- email - varchar(255), not null

База данных PostgreSQL, доступы (пропиши их в .env): psql -h localhost -U users_db_user -d users_db -p 5432, пароль tmp. Используй psycopg3 и SQLAlchemy Core. Используй асинхронный код для доступа к БД и в FastAPI. Используй типизированный код.

Работу с БД вынеси в слой репозитория. Для миграций используй Alembic.

Используй ruff в качестве линтера и убедись, что код успешно проходит его проверку.

Убедись, что все CRUD сервисы работает. Напиши также тесты (используй pytest) на эти веб-ручки, пусть эти тесты работают с тестовой БД users_db_test.

Не забудь написать README.md.

chebupelka "выполни задачу из файла task.md"
```

48 шагов и задача выполнена. Вполне большая комплексная задача. На минимальнейшем самописном агенте из меньше чем 100 строк  кода, и на локальной модели qwen3.6-35b-a3b в кванте Q4_K_XL (==покажи==), модель запущена на одной карточке RTX 3090 за 70 тыс рублей на авито. Вот так вот, дорогие друзья!

Давайте пробежимся теперь по коду агента. Первая задача — это получить промпт от пользователя. Он берётся из аргумента командной строки. Затем запускается агентный цикл, и в него передаётся этот промпт. Затем составляется список сообщений для модели, на начальном этапе он состоит из системного промпта и одного пользовательского промпта. В `max_turns` хранится ограничение на максимальное количество итераций агентного цикла. И затем запускается агентный цикл. Здесь он представлен не циклом `while True`, а циклом `for`, потому что у нас есть ограничение на количество итераций, и логичнее для этого использовать цикл `for`. Тело цикла сейчас большое, но это только потому, что я не стал декомпозировать его на маленькие функции, чтобы сократить общий размер агента и сделать код более явным.

В цикле у нас составляется запрос к большой языковой модели, в этот запрос передаётся весь контекст, то есть весь список `messages`. Сначала в этом списке только системный промпт и начальный промпт от пользователя, затем в него будут добавляться вызовы инструментов и ответы от инструментов (покажи `messages.append` ниже). Затем идёт запрос в LLM, печатается ответ от LLM, если вызовов инструментов в ответе нет, то цикл завершается и работа считается выполненной, иначе вызываются инструменты, их результаты добавляются в контекст, то есть в messages, и цикл уходит на следующую итерацию.

И такой совершенно простейший агент показывает отличные результаты. Те, кто думал, что в Claude Code, Codex и других подобных инструментах есть какая-то магия — я надеюсь, теперь вы понимаете, что это за магия. Магия из шести строк кода с агентским циклом, и всё, дальше просто обвязка вокруг этого, даже без которой агент отлично работает и решает поставленную многошаговую задачу.

Кстати, а есть ли у этого агента  доступ в интернет? Конечно. У него же есть доступ к bash, в котором есть `curl`. Давайте попросим его сделать что-то в интернете:

```shell
chebupelka "чо какие там статьи сегодня интересные есть, глянь на хабре"
```

Большая языковая модель уже знает ресурс Хабр, и агент сам вышел в Интернет, нашёл хабр, разобрался, где там лежат последние статьи и вывел нам summary по последним статьям. Кайф!

Никаких MCP, никаких сложных вещей — 99 строк кода и такие возможности. Чебупелька рулит!

Спасибо, что посмотрели! Остаёмся на связи, дооо следующих выпусков! Пока-пока! Пока-пока!