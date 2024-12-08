## Введение

Здоров котаны, пару дней назад вышло видео про оптимизацию вложенных циклов с примером на Python, которое вызвало бурное обсуждение в комментариях и бурную критику, в связи с чем я решил снять дополнение к этому видосу.

Сейчас мы наконец-то с вами во всём разберёмся!

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
print(dis.dis(big_outer_loop))
print(dis.dis(small_outer_loop))
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

Также звучали призывы вынести за рамки функций `range`, и вот мол результат будет совсем другой, это типа `range` вообще даёт нам такой результат. Давайте это сделаем.

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

## А если переписать на while?

А давайте перепишем цикл на `while`.

```python
import argparse
from functools import partial
import timeit

parser = argparse.ArgumentParser()
parser.add_argument("--big", type=int)
parser.add_argument("--small", type=int)
parser.add_argument("--timeit", default=100, type=int)
args = parser.parse_args()

def run_loop(outer_iterable, inner_iterable):
    outer_iterable = iter(outer_iterable)
    inner_iterable = iter(inner_iterable)
    outer_has_next = True
    inner_has_next = True
    while outer_has_next:
        try:
            next(outer_iterable)
        except StopIteration:
            outer_has_next = False
        while inner_has_next:
            try:
                next(inner_iterable)
            except StopIteration:
                inner_has_next = False

big = timeit.timeit(partial(run_loop, range(args.big), range(args.small)), number=args.timeit)
small = timeit.timeit(partial(run_loop, range(args.small), range(args.big)), number=args.timeit)
print(f"delta is {round(100*(big-small)/big)}%")
```

Здесь мы принимаем на вход две итерабельные сущности, получаем из них итераторы и итерируемся по ним.

Тесты!

```shell
$ python3.12 main_with_while.py --big 100 --small 5
delta is 8%

$ python3.12 main_with_while.py --big 500 --small 5
delta is 10%

$ python3.12 main_with_while.py --big 1500 --small 5
delta is 12%

$ python3.12 main_with_while.py --big 3000 --small 50
delta is 14%

$ python3.12 main_with_while.py --big 10000 --small 100
delta is 15%

$ python3.12 main_with_while.py --big 100000 --small 10
delta is 8%
```

Видим, что для `while` большой разницы в 30% получить не удаётся, однако всё равно меньший внешний цикл выигрывает.

## while работаем медленнее, чем for?

Давайте посмотрим разницу `while` и `for` в байткоде?

```python
from functools import partial
import timeit
from typing import Iterable

def for_loop(iterable: Iterable):
    for _ in iterable:
        pass


def while_loop(iterable: Iterable):
    iseq = iter(iterable)
    _loop = True
    while _loop:
        try:
            next(iseq)
        except StopIteration:
            _loop = False

print(timeit.timeit(partial(for_loop, range(1_000_000)), number=100))  # 2.1
print(timeit.timeit(partial(while_loop, range(1_000_000)), number=100))  # 3.4
```

 А чо так? Смотрим `dis` от `for`:
 
```text
  5           0 RESUME                   0

  6           2 LOAD_FAST                0 (iterable)
              4 GET_ITER
        >>    6 FOR_ITER                 2 (to 14)
             10 STORE_FAST               1 (_)

  7          12 JUMP_BACKWARD            4 (to 6)

  6     >>   14 END_FOR
             16 RETURN_CONST             0 (None)
```

Смотрим `dis` от `while`:

```text
 10           0 RESUME                   0

 11           2 LOAD_GLOBAL              1 (NULL + iter)
             12 LOAD_FAST                0 (iterable)
             14 CALL                     1
             22 STORE_FAST               1 (iseq)

 12          24 LOAD_CONST               1 (True)
             26 STORE_FAST               2 (_loop)

 13          28 LOAD_FAST                2 (_loop)
             30 POP_JUMP_IF_FALSE       16 (to 64)

 14     >>   32 NOP

 15          34 LOAD_GLOBAL              3 (NULL + next)
             44 LOAD_FAST                1 (iseq)
             46 CALL                     1
             54 POP_TOP

 13     >>   56 LOAD_FAST                2 (_loop)
             58 POP_JUMP_IF_FALSE        1 (to 62)
             60 JUMP_BACKWARD           15 (to 32)
        >>   62 RETURN_CONST             0 (None)
        >>   64 RETURN_CONST             0 (None)
        >>   66 PUSH_EXC_INFO

 16          68 LOAD_GLOBAL              4 (StopIteration)
             78 CHECK_EXC_MATCH
             80 POP_JUMP_IF_FALSE        5 (to 92)
             82 POP_TOP

 17          84 LOAD_CONST               2 (False)
             86 STORE_FAST               2 (_loop)
             88 POP_EXCEPT
             90 JUMP_BACKWARD           18 (to 56)

 16     >>   92 RERAISE                  0
        >>   94 COPY                     3
             96 POP_EXCEPT
             98 RERAISE                  1
ExceptionTable:
  34 to 54 -> 66 [0]
  66 to 86 -> 94 [1] lasti
  92 to 92 -> 94 [1] lasti
None
```

Как видим, здесь больше кода для `while`, однако это не значит, что именно поэтому он работает медленнее — как мы видели раньше, тупо повторение кода вместо использования цикла вообще работает быстрее.

Однако здесь дело в том, что for написан на C, а этот код интерпретируется и потому работает медленнее.

Давайте перепишем while на счетчиках вместо сложного кода с итератором:

```python
from functools import partial
import timeit

def for_loop(count: int):
    for _ in range(count):
        pass


def while_loop(count: int):
    counter = 0
    while counter < count:
        counter += 1

print(timeit.timeit(partial(for_loop, 1_000_000), number=100))  # 2.1
print(timeit.timeit(partial(while_loop, 1_000_000), number=100))  # 3.3
```

Результат аналогичный — `for` работает быстрее. Такие дела.

## Выводы

Ну а в общем — меньший внешний цикл работает быстрее, чем больший внешний цикл при прочих равных. Что значит при прочих равных? Это значит, что если нет причины итерироваться именно конкретным способом. Например, если мы реализуем в своём коде аналог CROSS JOIN из SQL, когда нам надо обработать все пары возможных значений из двух наборов данных. Понятно, что здесь нет разницы какой цикл ставить внешним — и тогда стоит внешним циклом поставить меньший цикл.

Например, в нашем продукте Salesbeat, это модуль доставки для интернет магазинов, который считает доставку, есть логика расчёта сроков и стоимости доставки по пунктам выдачи. И эти сроки и стоимости можно у нас настраивать, то есть магазин может давать свои наценки или скидки на стоимость доставки, а также изменять сроки доставки по разным службам доставки или даже конкретным пунктам выдачи.

Ну и там непростая логика, реализовывать её на уровне базы в данном случае не хотелось бы.

Получается, у нас есть набор пунктов выдачи и он может быть на несколько тысяч пунктов, и есть набор правил, он достигает нескольких десятков. В каком порядке это перебирать с логической точки зрения разницы нет — можно сначала по пунктам, потом по правилам, можно наоборот. Вот просто поменяв порядок циклов, можно добиться ускорения этого конкретного участка.

При этом понятно, что с точки зрения конечного результата это может не дать эффекта, то есть вот эта часть итерации ускорится, но логика, которая внутри вложенного цикла выполняется, может быть настолько тяжелой, что на конечное время выполнения сервиса это никак не повлияет. Потому что основная часть времени уходит именно на эту логику, а не на саму логику обслуживания циклов. Или на запросы к СУБД.

Давайте внутрь цикла добавим вызов некоторой функции, которая займёт время. Это функция находит простые чисел с помощью решета Эратосфена (ударение на *е*):

```python
import argparse
from functools import partial
import random
import time
import timeit

parser = argparse.ArgumentParser()
parser.add_argument("--big", type=int, required=True)
parser.add_argument("--small", type=int, required=True)
parser.add_argument("--timeit", default=100, type=int)
parser.add_argument("--with-logic", action="store_true")
args = parser.parse_args()

big_tuple = tuple(random.randint(1000, 10_000) for _ in range(args.big))
small_tuple = tuple(random.randint(1000, 10_000) for _ in range(args.small))

def sieve_of_eratosthenes(limit):
    primes = [True] * (limit + 1)
    p = 2
    while p * p <= limit:
        if primes[p]:
            for i in range(p * p, limit + 1, p):
                primes[i] = False
        p += 1
    return [p for p in range(2, limit + 1) if primes[p]]

def run_loop(outer_iterable, inner_iterable, with_logic):
    overall = 0
    for _ in outer_iterable:
        for _ in inner_iterable:
            overall += 1
            if with_logic:
                sieve_of_eratosthenes(50)
    
big = timeit.timeit(
    partial(run_loop, big_tuple, small_tuple, args.with_logic),
    number=args.timeit)
small = timeit.timeit(
    partial(run_loop, small_tuple, big_tuple, args.with_logic),
    number=args.timeit)
print(f"delta is {round(100*(big-small)/big)}%")
```

Запустим тесты:

```shell
$ python3.12 final_compare.py --big 100 --small 5
delta is 45%

$ python3.12 final_compare.py --big 100 --small 5 --with-logic
delta is 2%


$ python3.12 final_compare.py --big 500 --small 5
delta is 46%

$ python3.12 final_compare.py --big 500 --small 5  --with-logic
delta is 3%


$ python3.12 final_compare.py --big 1500 --small 5
delta is 47%

$ python3.12 final_compare.py --big 1500 --small 5 --with-logic
delta is 1%
```

То есть, конечно, это не то, что в большинстве практических сценариев драматически ускорит ваше приложение, там на 30%, ну нет, конечно. Это просто принцип, который можно знать и использовать при написании кода. Что-то, что не осложняет разработку, не ухудшает читаемость кода, не ухудшает поддерживаемость кода, не ухудшает расширяемость кода, но при этом может положительно сказаться на эффективности какой-то части программы.

Если упростить логику в цикле, то разница снова становится заметной:

```python
def run_loop(outer_iterable, inner_iterable, with_logic):
    overall = 0
    for _ in outer_iterable:
        for index in inner_iterable:
            overall += 1
            if with_logic:
                if index // 2:
                    some_tuple = (1, 2, 3)
                    overall -= len(some_tuple) + some_tuple[-1]
```

Это плохой код, тут большая вложенность и ничего непонятно — я привожу его просто для примера несложной с вычислительной точки зрения логики — проще, чем вычисление простых чисел.

Тесты:

```shell
$ python3.12 final_compare.py --big 100 --small 5 --with-logic
delta is 15%

$ python3.12 final_compare.py --big 500 --small 5  --with-logic
delta is 16%

$ python3.12 final_compare.py --big 1500 --small 5 --with-logic
delta is 6%
```

Такие дела. Пользуйтесь.

Спасибо, что посмотрели, надеюсь, что-то полезное узнали, до связи в следующем видео! Пока-пока:)

[[types/Сценарий]]