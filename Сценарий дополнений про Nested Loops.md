## Введение

Здоров котаны, вчера вышло видео про оптимизацию вложенных циклов с примером на Python, которое вызвало бурное обсуждение в комментариях и я решил снять дополнение к этому видосу.

Да, выводы, которые звучали во вчерашнем видосе, оказались поспешными и не всегда правильными, как мы сейчас убедимся.  Сейчас мы наконец-то с вами во всём разберёмся и сделаем прррравильные выводы!

// ну, скорее всего...

Посмотрим, как работают циклы, как они выглядят в дизассемблированном виде, как правильно считать итерации и как всё таки эффективно использовать вложенные циклы!

## О чём речь

Коротенько напомню, об чём речь. Вывод в видосе: когда у вас есть вложенные циклы, для эффективности имеет смысл цикл меньшего размера ставить наружным, а цикл большего размера ставить внутренним.

В видео демонстрировался совершенно великолепный код на питоняке:

```python
import random
import timeit

ROWS = 5
COLUMNS = 100
TIMEIT_ITERATIONS = 1_000

table = tuple(tuple(random.randint(1000, 10_000)
                    for _ in range(COLUMNS))
              for _ in range(ROWS))

def big_outer_loop():
    overall_sum = 0
    for column in range(COLUMNS):
        for row in range(ROWS):
            overall_sum += table[row][column]

def small_outer_loop():
    overall_sum = 0
    for row in range(ROWS):
        for column in range(COLUMNS):
            overall_sum += table[row][column]


small_outer_loop_time  = timeit.timeit(small_outer_loop, number=TIMEIT_ITERATIONS)
big_outer_loop_time = timeit.timeit(big_outer_loop, number=TIMEIT_ITERATIONS)

delta = round(100*(big_outer_loop_time - small_outer_loop_time) / big_outer_loop_time)
print(f"{delta}%")
```

Здесь есть таблица на 5 строк и 500 колонок и мы проходим эту таблицу либо сначала по колонкам, потом по строкам, либо сначала по строкам, потом по колонкам. И в моём тесте получалось, что эффективнее внешним циклом ставить тот, в котором меньше итераций, потому что это отрабатывало на 20-30% быстрее плюс-минус. Вот сейчас показывает 20%.

## Всё дело в обходе структуры!

В комментариях написали, что это у тебя оно так работает, потому что ты проходишь по структуре и эффективнее проходить по данным, которые лежат на диске или в памяти рядом. Конечно, последовательный доступ к рядом лежащим данным это хорошо, однако, в данном случае это не особо влияет на результаты, давайте в этом убедимся, вообще выбросив таблицу:

```python
import timeit

ROWS = 5
COLUMNS = 100
TIMEIT_ITERATIONS = 1_000

def big_outer_loop():
    overall_sum = 0
    for column in range(COLUMNS):
        for row in range(ROWS):
            overall_sum += 1

def small_outer_loop():
    overall_sum = 0
    for row in range(ROWS):
        for column in range(COLUMNS):
            overall_sum += 1

small_outer_loop_time  = timeit.timeit(small_outer_loop, number=TIMEIT_ITERATIONS)
big_outer_loop_time = timeit.timeit(big_outer_loop, number=TIMEIT_ITERATIONS)

delta = round(100*(big_outer_loop_time - small_outer_loop_time) / big_outer_loop_time)
print(f"{delta}%")
```

То, что вместо обхода таблицы выбрана задача инкремента какой-то переменной — это неважно. На практике любая другая задача здесь может быть. Понятно, что просто посчитать сумму можно проще, здесь это просто для примера как некая простая операция.

Разница получилась 29%.

Здесь может быть предположение, что умный компилятор Python схлопывает эти тупенькие циклы и сразу считает значение `overall_sum`, давайте это проверим. Кстати, в Python есть компилятор — он компилирует исходный код в байткод, который затем выполняется виртуальной машиной Python.  И у нас есть возможность посмотреть этот код байт-код в виде программы на языке ассемблера:

```python
import dis
dis.dis(big_outer_loop)
```

Повторюсь, что это не байт-код, связанный с конкретной системой команд конкретной архитектуры процессора, потому что его выполнять будет Python Virtual Machine, виртуальная машина Python.

Сохраним результаты дизассемблирования двух функций в 2 файла, открываем и убеждаемся, что они идентичны и отличаются только количеством итераций.

```text
  7           0 RESUME                   0

  8           2 LOAD_CONST               1 (0)
              4 STORE_FAST               0 (overall_sum)

  9           6 LOAD_GLOBAL              1 (NULL + range)
             16 LOAD_GLOBAL              2 (COLUMNS)
             26 CALL                     1
             34 GET_ITER
        >>   36 FOR_ITER                27 (to 94)
             40 STORE_FAST               1 (column)

 10          42 LOAD_GLOBAL              1 (NULL + range)
             52 LOAD_GLOBAL              4 (ROWS)
             62 CALL                     1
             70 GET_ITER
        >>   72 FOR_ITER                 7 (to 90)
             76 STORE_FAST               2 (row)

 11          78 LOAD_FAST                0 (overall_sum)
             80 LOAD_CONST               2 (1)
             82 BINARY_OP               13 (+=)
             86 STORE_FAST               0 (overall_sum)
             88 JUMP_BACKWARD            9 (to 72)

 10     >>   90 END_FOR
             92 JUMP_BACKWARD           29 (to 36)

  9     >>   94 END_FOR
             96 RETURN_CONST             0 (None)
```

Левая колонка показывает тут номер строки в Python-коде, средняя колонка показывает команду, правая колонка аргументы команды.

- `RESUME` (*ризююм*) — начало функции
- `LOAD_CONST` — загрузка в стек константы 0
- `STORE_FAST` — сохраняет значение с вершины стека в локальную переменную `overall_sum`
- `LOAD_GLOBAL` — загрузка глобалной функции `range`
- следующий `LOAD_GLOBAL` — загрузка значения `COLUMNS`
- `CALL` — вызов функции `range`
- `GET_ITER` и `FOR_ITER` это работа с итератором, которая тут не раскрывается
- `STORE_FAST` — сохраняет значение с вершины стека в локальную переменную `column`
- `LOAD_GLOBAL` — снова загружается `range` и `ROWS`, затем `CALL` вызывается  `range`, затем сохраняем значение с вершины стека в локальную переменную `row`
- и затем видим тело вложенного цикла — тут нет никакой оптимизации, просто инкремент значения переменной `overall_sum` с командой `BINARY_OP`.
-  Затем `JUMP_BACKWARD` для возврата к вложенному циклу и затем `JUMP_BACKWARD` для возврата ко внешнему циклу для следующей итерации.
- И затем `RETURN_CONST` это выход из функции, с возвратом `None` в данном случае.  

То есть делаем вывод, что никакой оптимизации сложения тут нет и внутрянка обоих циклов идентична. И также делаем вывод, что дело здесь не в том, как мы обходим структуру, а в чем-то другом, потому что мы убрали структуру, а результат тот же.

## Надо просто вынести range за функции!

Также звучали призывы вынести за рамки функций `range`. Давайте это сделаем.

```python
import timeit

ROWS = range(5)
COLUMNS = range(100)
TIMEIT_ITERATIONS = 1_000

def big_outer_loop():
    overall_sum = 0
    for column in COLUMNS:
        for row in ROWS:
            overall_sum += 1

def small_outer_loop():
    overall_sum = 0
    for row in ROWS:
        for column in COLUMNS:
            overall_sum += 1

small_outer_loop_time  = timeit.timeit(small_outer_loop, number=TIMEIT_ITERATIONS)
big_outer_loop_time = timeit.timeit(big_outer_loop, number=TIMEIT_ITERATIONS)

delta = round(100*(big_outer_loop_time - small_outer_loop_time) / big_outer_loop_time)
print(f"{delta}%")
```

Разница все равно в районе 20% остаётся. Изменился ли байткод?

Да, из него пропала очевидным образом загрузка `range` и вызов `range`. В остальном всё работает как и работало и, как показывают цифры, всё равно меньший внешний цикл работает лучше, чем больший внешний цикл.

Давайте заменим вызов генератора на просто кортеж:

```python
ROWS = tuple(range(5))
COLUMNS = tuple(range(100))
```

Разница получилась еще больше — 40%.

## Поговорим об итерациях

На этом мы копать не заканчиваем, но давайте теперь сделаем отступление вообще и циклах и итерациях. Много людей в комментариях написано мне, что я неправильно считаю итерации, складывая количество выполнений внешнего и внутреннего цикла, что количество итераций в обоих случаях 500.

Давайте поговорим о том, зачем вообще я считаю количество раз, которое крутится цикл, допустим, пока не будем называть это итерациями. Потому что мы тут измеряем производительность, а цикл несёт в себе накладные расходы, бесплатно как правило ничего не бывает.

```python
import dis
import timeit

def func1():
    overall = 0
    overall += 1
    overall += 1
    overall += 1
    overall += 1
    overall += 1
    overall += 1
    overall += 1
    overall += 1
    overall += 1
    overall += 1
    return overall

VALUES = tuple(range(10))

def func2():
    overall = 0
    for _ in VALUES:
        overall += 1
    return overall

print(func1(), timeit.timeit(func1, number=1000))
print(func2(), timeit.timeit(func2, number=1000))
```

Что, как думаете, какая функция быстрее будет? Есть ли разница вообще? В первой функции строк кода больше, и вообще это плохой код какой-то, наверняка он дольше будет? А насколько, как думаете?

Видим разницу примерно в 2 раза — функция без цикла работает быстрее. Щас возможно у кого-то открылось прозрение, но цикл не бесплатный!

Опять же, чтобы проверить, что в байт коде нет каких-то хитрых оптимизаций, посмотрим на него. Видим, что `func1` это просто повторение кода инкремента переменной. И видим, что `func2` это просто цикл. Никаких оптимизаций здесь компилятор не делает.

Так почему первый код, который тупо повторяет одну и ту же операцию без цикла, вдруг работает быстрее аж в 2 раза, там плюс-минус? Потому что есть цикл. Потому что он не бесплатный. Здесь не показывается реализация самого цикла FOR в байт-коде, но там в любом случае есть какой-то счетчик, который надо инициализировать, инкрементировать, проверять и так далее, а в случае с тупо перечислением операций без цикла этих всех операций делать не надо.

И именно поэтому я считаю количество сколько раз крутятся оба цикла, внешний и внутренний.

Теперь касательно того, что есть итерация.

Вот [пример цикла](https://metanit.com/assembler/arm64/2.9.php) на настоящем ассемблере, не внутри PVM:

```asm
.global _start

_start:
    mov x0, #0          // регистр X0 - условный счетчик
for_start:              // метка, на которую проецируется цикл
    cmp x0, #5         // сравниваем с некоторым пределом
    b.ge for_end         // условие - если счетчик больше или равен пределу, выход из цикла
    // выполняемые действия
    add x0, x0, 1       //  действия цикла - увеличение счетчика
    b for_start       // повторяем цикл
for_end:
// завершение программы
    mov x8, #93         // номер функции Linux для выхода из программы - 93
    svc 0               // вызываем функцию и выходим из программы
```

Что здесь итерация? Это всё между метками `for_start` и `for_end`. Всё это повторно выполняется при работе цикла. Постоянно код курсирует между этими метками. Это ли не есть итерация? И что же находится между этими метками? А находится там работа со счетчиком и сама полезная нагрузка цикла, оно же тело цикла, которое вы в питоне отделяете табуляцией. Но это всё итерация, не только отделенная табуляцией часть.

Если не хочется по какой-то причине называть это итерациями, и хочется почему-то назвать итерацией только часть кода, которая повторно выполняется при работе цикла, я в целом не против, назовите это операциями, вычислениями, я не знаю, фрикциями, ассимиляциями гиперпостранственного фокала социалистических измерений, пофик ваще. Суть не изменится, этот код многократно выполняется при работе цикла. Я вот по рабоче-крестьянски предпочту это назвать итерацией.

Давайте сделаем свою версию **генератора** `range` и посмотрим, сколько раз она вызовется в случае большего и меньше внешнего цикла:

```python
def my_range(limit):
    counter = 0
    while counter < limit:
        yield 1
        counter += 1
    my_range.called += counter

my_range.called = 0

for _ in my_range(100):
    for _ in my_range(5):
        pass

print(my_range.called)  # 600

my_range.called = 0

for _ in my_range(5):
    for _ in my_range(100):
        pass

print(my_range.called)  # 505
```

Почему, если итераций 500, у нас выводится в одном случае 600, в другом, 505? Да потому что общее число итераций не 500. В одном случае **генератор** суммарно вызвался 600 раз, во втором случае он суммарно вызвался 505 раз.

Но если чо — ассимиляциями гиперпостранственного фокала социалистических измерений это тоже нормальное название, я не против.

## Упрощаем код

Ладно, давайте упростим наш код, чтобы с ним было проще играться.

```python
import argparse
from functools import partial
import random
import time
import timeit

parser = argparse.ArgumentParser()
parser.add_argument("--big", type=int)
parser.add_argument("--small", type=int)
parser.add_argument("--timeit", default=100, type=int)
args = parser.parse_args()

big_tuple = tuple(random.randint(1000, 10_000) for _ in range(args.big))
small_tuple = tuple(random.randint(1000, 10_000) for _ in range(args.small))

def run_loop(outer_iterable, inner_iterable):
    overall = 0
    for _ in outer_iterable:
        for _ in inner_iterable:
            overall += 1

big = timeit.timeit(partial(run_loop, big_tuple, small_tuple), number=args.timeit)
small = timeit.timeit(partial(run_loop, small_tuple, big_tuple), number=args.timeit)
print(f"delta is {round(100*(big-small)/big)}%")
```

Здесь я решил вынести наружу значения ограничения большего и меньшего циклов, чтобы они принимались из аргументов командной строки при запуске скрипта. Также оттуда передаётся количество запусков для `timeit` с дефолтным значением 100.

И вместо двух функций сделал одну, которая просто вызывается с разными параметрами. 

## Адовые тесты!

Ну так вот. Всё мы значит выяснили, где чо как работает, где какие итерации, что эффективнее и почему. Быгыгы:)

Давайте протестируем наш код с разными входными данными.

```shell
$ python3.12 main_with_for.py --big 100 --small 5
delta is 47%

$ python3.12 main_with_for.py --big 500 --small 5
delta is 45%

$ python3.12 main_with_for.py --big 1500 --small 5
delta is 48%

$ python3.12 main_with_for.py --big 3000 --small 50
delta is 7%

$ python3.12 main_with_for.py --big 10000 --small 100
delta is 2%

$ python3.12 main_with_for.py --big 100000 --small 10
delta is 29%
```

Что мы видим — мы видим, что на каких-то значениях меньший внешний цикл эффективнее на 47%, на каких-то на 30%, а где-то и почти неважно, какой цикл внешний.

Какой отсюда можно сделать вывод?

При прочих равных в 3.12 CPython если вы будете ставить меньший цикл внешним, вы или увеличите общую эффективность, или как минимум её не уменьшите. Конечно, при условии, что код в циклах независим от порядка итерирования.

Ну а для знатоков ассимиляций гиперпостранственного фокала социалистических измерений — вот вам задачка.

Я перепишу циклы на while, чтобы сделать работу со счетчиками явными.

```python
import argparse
from functools import partial
import random
import timeit

parser = argparse.ArgumentParser()
parser.add_argument("--big", type=int)
parser.add_argument("--small", type=int)
parser.add_argument("--timeit", default=100, type=int)
args = parser.parse_args()

big_tuple = tuple(random.randint(1000, 10_000) for _ in range(args.big))
small_tuple = tuple(random.randint(1000, 10_000) for _ in range(args.small))

def run_loop(outer_iterable, inner_iterable):
    i = 0
    overall_sum = 0
    while i < len(outer_iterable):
        j = 0
        while j < len(inner_iterable):
            overall_sum += 1
            j += 1
        i += 1
    return overall_sum
    
big = timeit.timeit(partial(run_loop, big_tuple, small_tuple), number=args.timeit)
small = timeit.timeit(partial(run_loop, small_tuple, big_tuple), number=args.timeit)
print(f"delta is {round(100*(big-small)/big)}%")
```

Запускаем:

```shell
$ python3.12 main_with_while.py --big 100 --small 5
delta is 17%

$ python3.12 main_with_while.py --big 500 --small 5
delta is 9%

$ python3.12 main_with_while.py --big 1500 --small 5
delta is 5%

$ python3.12 main_with_while.py --big 3000 --small 50
delta is -27%

$ python3.12 main_with_while.py --big 10000 --small 100
delta is -30%

$ python3.12 main_with_while.py --big 100000 --small 10
delta is -29%
```

Интереееесно! А если убрать кортежи вообще?

```python
import argparse
from functools import partial
import timeit

parser = argparse.ArgumentParser()
parser.add_argument("--big", type=int)
parser.add_argument("--small", type=int)
parser.add_argument("--timeit", default=100, type=int)
args = parser.parse_args()

def run_loop(outer_iterations, inner_iterations):
    i = 0
    overall_sum = 0
    while i < outer_iterations:
        j = 0
        while j < inner_iterations:
            overall_sum += 1
            j += 1
        i += 1
    return overall_sum
    
big = timeit.timeit(partial(run_loop, args.big, args.small), number=args.timeit)
small = timeit.timeit(partial(run_loop, args.small, args.big), number=args.timeit)
print(f"delta is {round(100*(big-small)/big)}%")
```

Получаем аналогичные результаты:

```shell
$ python3.12 main_with_while.py --big 100 --small 5
delta is 19%

$ python3.12 main_with_while.py --big 500 --small 5
delta is 14%

$ python3.12 main_with_while.py --big 1500 --small 5
delta is 7%

$ python3.12 main_with_while.py --big 3000 --small 50
delta is -27%

$ python3.12 main_with_while.py --big 10000 --small 100
delta is -31%

$ python3.12 main_with_while.py --big 100000 --small 10
delta is -20%
```

Итак, призываю вас, добро пожаловать в комментарии! Шо там по кешам процессора, спотыкающимся электронам, специальным закладкам Гвидо ван Россума в коде CPython и другим причинам такого поведения.

Спасибо, что посмотрели, надеюсь, что-то полезное узнали, до связи в следующем видео! Пока-пока:)

[[Сценарий]]