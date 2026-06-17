Ооо, здоров, котаны, огненный материал сегодня для вас подготовил! Сам бы с кайфом хотел такой посмотреть некоторое время назад:)

Напишем сегодня свой агент для разработки, такой свой минимальный claude code за 100 строк кода. Вообще серьёзно.

Поговорим про agent loop и про то, как под капотом работают все более-менее автономные ИИ-агенты, например, Claude Code, Codex, OpenCode, pi, OpenClaw, Hermes и прочие (==покажи их названия==). Это очень интересно с одной стороны, и удивительно легко и просто с другой стороны. Дело в том, что ключевая функциональность всех этих агентов, вот ядро, может быть реализована в 6 строках кода. Я серьёзно, вот эти 6 строк на питоне:

```python
while True:
    response = callLLM(messages)
    if response.toolCalls:
        messages.append(executeTools(response.toolCalls))
    else:
        return response
```

Мы дальше подробнее разберём этот код и напишем на его основе своего автономного кодинг ассистента. Ну и если вы хотите разрабатываете своих автономных агентов, необязательно для написания кода, и не понимаете, как они работают, то этот материал ровно чётенько для вас. Потому что цель у агента может быть любой, это небязательно написание кода, а принципы работы агента будут ровно те же самые.

Короче.  Что такое ИИ-агент? Определений много, все они подчёркивают разные особенности и смотрят на агентов с разных сторон, но давайте назовём по-простому ИИ-агентом связку большой языковой модели и доступных ей инструментов. То есть агент вызывает большую языковую модель через API и передаёт ей помимо промпта ещё набор инструментов, есть функций с их названиями и описаниями параметров. И модель в ответе может просить агента вызвать эти инструменты с такими-то аргументами. Агент вызывает инструменты и передаёт результат их вызова обратно модели. И в ответ модель может или вернуть конечный ответ пользователю или снова вернуть просьбу вызвать какой-то инструмент и так далее, пока цель не будет достигнута и задача не будет решена.

Вот это и есть тот самый aget loop (==покажи agent loop==), агентский цикл, код которого я уже показывал:

```python
while True:
    response = callLLM(messages)
    if response.toolCalls:
        messages.append(executeTools(response.toolCalls))
    else:
        return response
```

У нас тут есть бесконечный цикл `while True`. Структура `messages` это список всех сообщений в модель и от модели, то есть история контекста. Функция `callLLM` отвечает за вызов модели с переданным ей контекстом. Эта функция возвращает ответ от модели, в этом ответе может быть просьба вызвать инструмент или может быть просто ответ. Если есть просьба вызвать инструмент, то этот инструмент вызывается в функции `executeTools` и результат от инструмента тоже добавляется в контекст. И цикл уходит на следующую итерацию. Выход из цикла происходит тогда, когда модель прекратила возвращать просьбу вызвать инструмент, это считается либо решённой задачей, либо ситуацией, когда необходимо подключиться человеку, то есть нужно какое-то действие или уточнение от человека.

На практике этот цикл, конечно, необязательно будет бесконечным, и имеет смысл установить какое-то разумное ограничение на максимальное количество итераций.

Иии — вот и всё. По сути так и работают все автономные ИИ-агенты. Конечно, вокруг этого накручивается дополнительная логика на проверки безопасности, на сжатие контекста, когда он приближается к ограничению контекстного окна, и так далее, но суть, ядро — ровно такая, вот эти 7 строк кода, которые в том или ином виде представлены во всех этих ИИ-агентах.

Ну что, давайте бахнем минимального кодинг ассистента. На питончике. Который будет коммуницировать с локальной LLM-моделью и шарашить нам проекты. Но для начала нам надо подумать, как наш агент будет коммуницировать с внешним миром? Например, читать файлы, записывать файлы, искать файлы, запускать проекты для их проверки? А очень просто: давайте дадим ему доступ в bash, вот ровно один инструмент. Используя bash, он сможет делать все эти и массу других великолепных операций.

Если вдруг кто не понимает, что за bash такой интересный, то я имею в виду доступ к полноценной консоли linux, unix, mac os. Ну, или консоли Windows, если вы предпочитаете Windows, но про windows мы тут не говорим. Да, безусловно, это небезопасно, давать доступ ко всей системе, но вы можете запускать агента, например, в изолированном контейнере, в изолированной виртуалке или где угодно ещё. Или накрутить поверх нашего минимального кода логику разных проверок.

Итак, а вот и наш агент (`/home/sterx/minimal-coding-harness`):

```python
"""Minimal coding agent loop — one tool: bash."""
import json
import subprocess

from openai import OpenAI

# ── LLM config ───────────────────────────────────────────────
BASE_URL = "http://192.168.2.66:1234/v1"
API_KEY = "sk-lm-8lxtW0iY:g7ImIC5vURnv2nJzxG5b"
MODEL   = "qwen3.6-35b-a3b"

client = OpenAI(base_url=BASE_URL, api_key=API_KEY)

# ── Tool definition (only bash) ──────────────────────────────
TOOLS = [
    {
        "type": "function",
        "function": {
            "name": "bash",
            "description": "Execute a shell command and return the output.",
            "parameters": {
                "type": "object",
                "properties": {
                    "command": {
                        "type": "string",
                        "description": "The bash command to execute.",
                    },
                },
                "required": ["command"],
            },
        },
    }
]

SYSTEM_PROMPT = """\
You are a coding agent. Your job is to help the user with programming tasks.

You have access to ONE tool: `bash` — which executes shell commands and returns stdout/stderr.

Workflow:
1. Plan what needs to be done.
2. Use `bash` to read files, run commands, write code, etc.
3. After gathering enough information or completing the task, give your final answer in natural language.
4. To finish, reply with a regular message (no tool call).

Be concise. Explain what you're doing before each command."""


def run_bash(command: str) -> str:
    """Execute a shell command and return its output."""
    try:
        result = subprocess.run(
            command, shell=True, capture_output=True, text=True, timeout=120
        )
        out = result.stdout
        if result.stderr:
            out += "\nSTDERR:\n" + result.stderr
        return f"Exit code: {result.returncode}\n{out}"
    except subprocess.TimeoutExpired:
        return "Error: command timed out after 120s"


# ── Tool dispatcher ──────────────────────────────────────────
TOOLS_MAP = {"bash": run_bash}


def call_tool(name: str, arguments: dict) -> str:
    func = TOOLS_MAP.get(name)
    if not func:
        return f"Error: unknown tool '{name}'"
    try:
        return func(**arguments)
    except Exception as e:
        return f"Error calling {name}: {e}"


# ── Agent loop ───────────────────────────────────────────────
def agent_loop(user_message: str):
    messages = [
        {"role": "system", "content": SYSTEM_PROMPT},
        {"role": "user",   "content": user_message},
    ]

    max_turns = 20  # safety limit

    for turn in range(1, max_turns + 1):
        print(f"\n{'='*60}")
        print(f"🔄 Turn {turn}")
        print(f"{'='*60}")

        resp = client.chat.completions.create(
            model=MODEL,
            messages=messages,
            tools=TOOLS,
            tool_choice="auto",
            temperature=0.1,
            max_tokens=4096,
        )

        choice = resp.choices[0]
        msg = choice.message

        # Show assistant's text (if any)
        content = msg.content.strip() if msg.content else ""
        has_tool_calls = bool(getattr(msg, "tool_calls", None))

        if content:
            print(f"\n🤖 {content}")

        # Check for tool calls
        tool_calls = getattr(msg, "tool_calls", None) or []
        if not tool_calls:
            if not content:
                print("(no text output)")
            print("✅ Agent finished.")
            return msg.content

        # Process each tool call — only add blank line after 🤖 block
        prefix = ""
        if has_tool_calls and content:
            prefix = "\n"  # blank line between text and tools

        for tc in tool_calls:
            func_name  = tc.function.name
            func_args  = json.loads(tc.function.arguments)
            tc_id      = tc.id

            print(f"{prefix}🔧 Tool: {func_name}({json.dumps(func_args, ensure_ascii=False)})")

            result = call_tool(func_name, func_args)
            print(f"   → {result[:500]}{'...' if len(result)>500 else ''}")

            messages.append(
                {"role": "assistant", "content": None, "tool_calls": [tc]}
            )
            messages.append({
                "role": "tool",
                "tool_call_id": tc_id,
                "content": result,
            })

    print("\n⚠️  Max turns reached. Stopping.")


# ── CLI entry point ──────────────────────────────────────────
if __name__ == "__main__":
    import sys

    if len(sys.argv) > 1:
        prompt = " ".join(sys.argv[1:])
    else:
        print("Minimal Coding Agent Loop")
        print("Type your coding task (Ctrl-D to submit):\n")
        prompt = ""
        while True:
            try:
                line = input()
                if prompt:
                    prompt += "\n" + line
                else:
                    prompt = line
            except EOFError:
                break

    if not prompt.strip():
        print("No task provided. Exiting.")
        sys.exit(1)

    agent_loop(prompt)
```