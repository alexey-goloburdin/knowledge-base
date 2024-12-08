Looping over something iterable — `for` is faster:

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

Looping with `N` iterations — `for` is faster:

```python
from functools import partial
import timeit

def for_loop(end: int):
    cnt = 0
    for i in range(end):
        cnt += 1

def while_loop(end: int):
    i = 0
    cnt = 0
    while i < end:
        i += 1
        cnt += 1

print(timeit.timeit(partial(for_loop, 1_000_000), number=10))    # 0.4459s
print(timeit.timeit(partial(while_loop, 1_000_000), number=10))  # 0.5638s
```

Use cases for use `while`:

- `while True` — for daemons, for example, or other cases, when you want manually exit from loop with `break`
- `while file.has_data()` or something same

[[Python]]