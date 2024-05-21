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

[[Python]]