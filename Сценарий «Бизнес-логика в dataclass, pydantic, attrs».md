===добавь про курс!!!===

Здоров, котаны! 

А вот пишете ли вы бизнес-логику в датаклассах? А в pydantic-моделях? А может быть в attrs-классах? Или вы пишете БЛ в своих обычных классах? М? Как вы делаете?

Я хочу сказать, что в большинстве случаев лучший выбор для написания БЛ это простые обычные классы, не датаклассы, не pydantic и не attrs-классы. Сейчас поговорим, почему так.

Итак, как мы все знаем, в питоне есть много ништяков. Вот, например, есть обычные классы, в которых можно хранить данные и их обрабатывать. В этом вообще суть ООП — в том, чтобы иметь возможность объединить данные и методы, их обрабатывающие, вместе, в класс.  Чтобы эта логика была инкапсулирована в одно место, а не размазана по всему проекту, и чтобы у этого класса можно было сделать удобный красивый маленький API из публичных полей и методов, а детали реализации были скрыты.

Как эта цель достигается с обычными классами? Собственно просто — в методе `__init__` прописываются все поля, которые будут у экземпляра этого класса, создаются публичные и непубличные поля и методы:

```python
from datetime import datetime, timedelta
from enum import StrEnum


class MeetingStatus(StrEnum):
    DRAFT = "draft"
    SCHEDULED = "scheduled"
    CONFIRMED = "confirmed"
    CANCELLED = "cancelled"


class Meeting:
    # Бизнес-правила
    _MIN_DURATION = timedelta(minutes=15)
    _MAX_DURATION = timedelta(hours=3)
    _RESCHEDULE_DEADLINE = timedelta(hours=2)  # нельзя переносить < чем за 2ч
    _CONFIRM_DEADLINE = timedelta(
        hours=24
    )  # подтверждение возможно не позднее чем за 24ч
    _QUORUM_RATIO = 0.6  # нужно 60% подтверждений
    _MAX_PARTICIPANTS = 50

    def __init__(
        self,
        *,
        title: str,
        starts_at: datetime,
        ends_at: datetime,
        host_id: str,
        participant_ids: set[str],
        created_at: datetime | None = None,
    ):
        self._title = self._validate_title(title)
        self._host_id = host_id
        self._participant_ids = self._validate_participants(
            host_id, participant_ids
        )

        self._created_at = created_at or datetime.now()
        self._starts_at, self._ends_at = self._validate_time_range(
            starts_at, ends_at
        )

        self._status: MeetingStatus = MeetingStatus.SCHEDULED
        self._cancel_reason: str | None = None

        # состояние подтверждений
        self._confirmed_by: set[str] = set()
        self._declined_by: set[str] = set()

    # Поля только на чтение
    @property
    def title(self) -> str:
        return self._title

    @property
    def status(self) -> MeetingStatus:
        return self._status

    @property
    def starts_at(self) -> datetime:
        return self._starts_at

    @property
    def ends_at(self) -> datetime:
        return self._ends_at

    @property
    def host_id(self) -> str:
        return self._host_id

    @property
    def participant_ids(self) -> frozenset[str]:
        return frozenset(self._participant_ids)

    @property
    def confirmed_by(self) -> frozenset[str]:
        return frozenset(self._confirmed_by)

    # Поведение
    def rename(self, new_title: str) -> None:
        self._ensure_active()
        self._title = self._validate_title(new_title)

    def invite(self, participant_id: str) -> None:
        self._ensure_active()
        if participant_id == self._host_id:
            raise ValueError("Host cannot be invited as a participant.")
        if participant_id in self._participant_ids:
            return
        self._participant_ids.add(participant_id)
        # при изменении состава — подтверждения участия сбросываем
        self._reset_confirmations()

    def remove_participant(self, participant_id: str) -> None:
        self._ensure_active()
        if participant_id not in self._participant_ids:
            return
        self._participant_ids.remove(participant_id)
        self._confirmed_by.discard(participant_id)
        self._declined_by.discard(participant_id)

        if len(self._participant_ids) == 0:
            self.cancel("No participants left")

    def confirm(self, user_id: str, *, now: datetime | None = None) -> None:
        now = now or datetime.now()
        self._ensure_active()

        if user_id != self._host_id and user_id not in self._participant_ids:
            raise ValueError("Only host or invited participants can confirm.")

        if self._starts_at - now < self._CONFIRM_DEADLINE:
            raise ValueError(
                "Too late to confirm: confirmation deadline passed."
            )

        self._declined_by.discard(user_id)
        self._confirmed_by.add(user_id)

        if self._has_quorum():
            self._status = MeetingStatus.CONFIRMED

    def decline(self, user_id: str) -> None:
        self._ensure_active()
        if user_id != self._host_id and user_id not in self._participant_ids:
            raise ValueError("Only host or invited participants can decline.")

        self._confirmed_by.discard(user_id)
        self._declined_by.add(user_id)

        # если уже confirmed — может стать обратно scheduled,
        # если кворум потерян
        if self._status == MeetingStatus.CONFIRMED and not self._has_quorum():
            self._status = MeetingStatus.SCHEDULED

    def reschedule(
        self,
        *,
        new_starts_at: datetime,
        new_ends_at: datetime,
        requested_by: str,
        now: datetime | None = None,
    ) -> None:
        now = now or datetime.now()

        self._ensure_active()

        if requested_by != self._host_id:
            raise ValueError("Only host can reschedule a meeting.")

        if self._starts_at - now < self._RESCHEDULE_DEADLINE:
            raise ValueError(
                "Too late to reschedule: reschedule deadline passed."
            )

        new_starts_at, new_ends_at = self._validate_time_range(
            new_starts_at, new_ends_at
        )

        if new_starts_at == self._starts_at and new_ends_at == self._ends_at:
            return

        self._starts_at = new_starts_at
        self._ends_at = new_ends_at

        # перенос сбрасывает подтверждения, статус откатываем
        self._reset_confirmations()
        self._status = MeetingStatus.SCHEDULED

    def cancel(self, reason: str) -> None:
        if self._status == MeetingStatus.CANCELLED:
            return
        self._status = MeetingStatus.CANCELLED
        self._cancel_reason = reason
        self._reset_confirmations()

    def _ensure_active(self) -> None:
        if self._status == MeetingStatus.CANCELLED:
            raise ValueError("Meeting is cancelled.")

    def _validate_title(self, title: str) -> str:
        title = (title or "").strip()
        if len(title) < 3:
            raise ValueError("Title too short.")
        if len(title) > 120:
            raise ValueError("Title too long.")
        return title

    def _validate_participants(
        self, host_id: str, participant_ids: set[str]
    ) -> set[str]:
        if not host_id:
            raise ValueError("Host is required.")
        participant_ids = set(participant_ids or set())
        participant_ids.discard(host_id)
        if len(participant_ids) > self._MAX_PARTICIPANTS:
            raise ValueError("Too many participants.")
        return participant_ids

    def _validate_time_range(
        self, starts_at: datetime, ends_at: datetime
    ) -> tuple[datetime, datetime]:
        if starts_at >= ends_at:
            raise ValueError("Meeting start must be earlier than end.")
        duration = ends_at - starts_at

        if duration < self._MIN_DURATION:
            raise ValueError("Meeting duration is too short.")
        if duration > self._MAX_DURATION:
            raise ValueError("Meeting duration is too long.")
        return starts_at, ends_at

    def _reset_confirmations(self) -> None:
        self._confirmed_by.clear()
        self._declined_by.clear()

    def _has_quorum(self) -> bool:
        # считаем всех, кто может подтвердить: хост + участники
        eligible = 1 + len(self._participant_ids)
        confirmed = len(self._confirmed_by)
        return confirmed / eligible >= self._QUORUM_RATIO

    def __repr__(self) -> str:
        return (
            f"Meeting(title={self._title!r}, status={self._status.value!r}, "
            f"starts_at={self._starts_at.isoformat()}, "
            f"ends_at={self._ends_at.isoformat()}, "
            f"host_id={self._host_id!r}, "
            f"participants={len(self._participant_ids)})"
        )
```

Это класс встречи, который может быть использован в планировщике встреч, как в календаре в больших компаниях, где бронируют встречи и приглашают на эти встречи участников.

Этот класс управляет доменной логикой — проверяет, что дата окончания встречи идёт **после** даты начала встречи, а не **до**, проверяет минимальную и максимальную продолжительность встречи в соответствии с бизнес-правилами, проверяет количество участников встречи, и переводит статус встречи в отменённую или подтверждённую встречу в зависимости от количества участников, подтвердивших, что они придут.

Это обычный классический ООП-шный подход, подход объектно-ориентированного программирования. В чём он заключается? Он заключается, ещё раз, в том, что данные и методы, обрабатывающие эти данные, находятся в одном месте, в классе. Соответственно этими данными и методами легче управлять, плюс есть удобный понятный небольшой публичный API, который нужен клиентам этого класса. Клиенты этого класса не знают деталей реализации, но они могут создать встречу, перенести встречу, добавить или удалить участников, и добавить подтверждение или отклонение участия во встрече какого-то конкретного приглашенного человека.

Обратите внимание — у нас тут всё хорошо защищено. Например, мы не можем вот так взять и случайно записать дату окончания встречи меньше, чем дату начала встречи.  Потому что у нас нет публичного доступного к изменению поля дата начала встречи и такого же поля дата окончания встречи. Мы можем менять эти поля только через методы — а эти методы проверят корректность этих полей.

Таким образом, сам класс внутри себя поддерживает свои инварианты. Инвариант это то, что всегда должно выполняться. Например, дата окончания встречи всегда должна быть после даты начала встречи. Тут  вообще много инвариантов — установлены минимальная и максимальная продолжительность встречи, максимальное количество участников встречи и так далее.

В то же время нам тут пришлось самим делать метод `__init__`, самим делать метод `__repr__`, мы не получили понятного списка полей экземпляра класса, они задаются в `__init__`, но это не очень наглядно.

Так вот — почему бы нам просто не взять и не воспользоваться датаклассом? У нас же есть в питоне датаклассы, которые автоматически генерируют для нас пачку удобных методов и вроде как мы получаем удобство и меньше кода самим набирать.

```python
from __future__ import annotations

from dataclasses import dataclass, field
from datetime import datetime, timedelta
from enum import Enum


class MeetingStatus(str, Enum):
    DRAFT = "draft"
    SCHEDULED = "scheduled"
    CONFIRMED = "confirmed"
    CANCELLED = "cancelled"


@dataclass
class MeetingDC:
    title: str
    starts_at: datetime
    ends_at: datetime
    host_id: str
    participant_ids: set[str] = field(default_factory=set)

    status: MeetingStatus = MeetingStatus.SCHEDULED
    cancel_reason: str | None = None

    confirmed_by: set[str] = field(default_factory=set)
    declined_by: set[str] = field(default_factory=set)

    # “константы” — тоже публичны, их тоже можно случайно менять
    MIN_DURATION: timedelta = timedelta(minutes=15)
    MAX_DURATION: timedelta = timedelta(hours=3)
    RESCHEDULE_DEADLINE: timedelta = timedelta(hours=2)
    CONFIRM_DEADLINE: timedelta = timedelta(hours=24)
    QUORUM_RATIO: float = 0.6

    def __post_init__(self) -> None:
        # Да, можно проверить инварианты тут…
        self.title = (self.title or "").strip()
        if len(self.title) < 3:
            raise ValueError("Title too short.")
        if self.starts_at >= self.ends_at:
            raise ValueError("Meeting start must be earlier than end.")

        duration = self.ends_at - self.starts_at
        if duration < self.MIN_DURATION:
            raise ValueError("Meeting duration is too short.")
        if duration > self.MAX_DURATION:
            raise ValueError("Meeting duration is too long.")

        self.participant_ids.discard(self.host_id)

    # доменные методы ниже...
```

Тут удобненько перечислены все поля экземпляра в одном месте наглядно, тут нам не надо задавать `__init__` и `__repr__` самим, потому что dataclass сделает это за нас. Ноооо... штука в том, что с архитектурной точки зрения и с точки зрения правильного ООП, мы создали себе проблем. Например, сейчас клиентский код может напрямую писать во все поля, они же публичные и все их видят и могут изменять:

```python
m = MeetingDC(
    title="Daily",
    starts_at=datetime(2026, 1, 10, 10, 0),
    ends_at=datetime(2026, 1, 10, 10, 30),
    host_id="u1",
    participant_ids={"u2", "u3"},
)

# встреча "закончилась" до начала
m.ends_at = m.starts_at - timedelta(hours=1)   

# перенос встречи без сброса подтверждений
m.starts_at = datetime(2026, 1, 10, 9, 0)

# делает встречу "confirmed" без кворума
m.status = MeetingStatus.CONFIRMED

# подкручивает правило кворума в рантайме
m.QUORUM_RATIO = 0.01
```

И мы тут не обращаемся к полям, которые начинаются с символа подчеркивания. Мы обращаемся только к публичным полям. Класс позволяет нам это делать. По сути этот класс уже не поддерживает внутри себя инварианты — уже и дата окончания встречи может быть раньше даты начала встречи, и вообще всё может быть в расколбашенном состоянии.

Понимаете проблему? В чём она заключается? Она заключается в том, что любой клиент класса имеет доступ на запись во внутренние поля. Это плохо, потому что клиенты класса могут сломать его внутреннее поведение.

Ещё это плохо, потому что внутреннее состояние класса публично видно клиенту класса и клиент класса видит в том числе детали реализации класса, что плохо. Во внутренних полях класс может хранить данные, которые относятся к деталям реализации, и открывать их публично клиентам класса не нужно. dataclass же нас к этому подталкивает. Таким образом, dataclass не даёт нам нормально инкапсулировать логику в класс, нормально скрыть детали реализации, нормально защитить инварианты и тд и тд и тд.

Можно ли как-то с этим бороться в случае использования датакласса? Можно, назвав поля экземпляра с символа подчёркивания:

```python
from dataclasses import dataclass

@dataclass
class Something:
    _field1: int
    _field2: int


s = Something(1, 2)
print(s)
# Something(_field1=1, _field2=2)
print(s._field1)
# 1
```

Это работает на уровне соглашения, что если поля и методы начинаются с символа подчёркивания, то они не являются частью публичного API класса и к ним не рекомендуется обращаться, а если клиент класса всё же это делает, обращается к ним, то он это делает на свой страх и риск и берёт на себя весь груз ответственности за свои действия. Сломал — сам виноват. Но случайно сломать не получится, потому что случайно обратиться к непубличному API нельзя.

При этом это всё равно кривой способ — например, я не смогу передать в инициализатор названия полей без подчёркивания!

```python
from dataclasses import dataclass

@dataclass
class Something:
    _field1: int
    _field2: int


s = Something(field1=1, field2=2)
```

Это, очевидно, не работает, так как таких полей в классе нет, есть только поля, которые начинаются с символа подчёркивания.

Да, это можно побороть как-то так:

```python
from dataclasses import dataclass

@dataclass(init=False)
class Something:
    _field1: int
    _field2: int

    def __init__(self, field1: int, field2: int):
        self._field1 = field1
        self._field2 = field2
```


Но тут уже мы получаем меньше преимуществ от датакласса, так как сами реализуем `__init__` с ручной простановкой значений полям. Плюс `__repr__` выводит имена с подчёркиваниями, что совершенно тупо:

```python
print(Something(field1=100, field2=200))
# Something(_field1=100, _field2=200)
```

То есть это всё конкретно костыли. Можно переназначить ещё и `__repr__`, но тогда преимуществ от датакласса ещё меньше.

Так вот давайте подумаем — почему так? Почему при использовании для бизнес-логики датаклассов появляется столько проблем?

Потому что давайте-ка мы прочитаем ещё раз слово `dataclass`. Что оно значит? Оно значит, внезапно, класс данных. Это просто контейнер для данных. Это не класс для БЛ. Это просто контейнер для данных с прописанной структурой и всё. И для таких сценариев использования датаклассы отлично подходят.

Например, для Value Objects датаклассы отлично подходят, это неизменяемые объекты, у которых нет идентичности и которые сравниваются по значениям полей. Например, объект температура:

```python
from dataclasses import dataclass
from enum import Enum
from typing import Self


class TemperatureUnit(str, Enum):
    CELSIUS = "C"
    FAHRENHEIT = "F"


@dataclass(frozen=True, slots=True)
class Temperature:
    _celsius: float

    @staticmethod
    def from_celsius(value: float) -> Self:
        Temperature._validate_celsius(value)
        return Temperature(value)

    @staticmethod
    def from_fahrenheit(value: float) -> Self:
        celsius = (value - 32) * 5 / 9
        Temperature._validate_celsius(celsius)
        return Temperature(celsius)

    @property
    def celsius(self) -> float:
        return self._celsius

    @property
    def fahrenheit(self) -> float:
        return self._celsius * 9 / 5 + 32
        
    def __post_init__(self):
        self._validate_celsius(self._celsius)

    def is_below_freezing(self) -> bool:
        return self._celsius < 0

    def add(self, delta: "TemperatureDelta") -> "Temperature":
        return Temperature.from_celsius(self._celsius + delta.celsius)

    @staticmethod
    def _validate_celsius(value: float) -> None:
        if value < -273.15:
            raise ValueError("Temperature cannot be below absolute zero")
```

Это Value Object. У этого объекта нет идентичности, то есть сравнение двух объектов осуществляется строго по значению полей, в данном случае по значению одного поля:

```python
print(
    Temperature.from_celsius(100) == Temperature.from_celsius(100)
)
# True
```

И эти объекты неизменяемы. За это отвечает параметр датакласса `frozen=True`. А параметр `slots=True` отвечает за более эффективную работу такого датакласса. Да, в таком классе может быть немного логики, но это несложная простая логика, поддерживающая инварианты, например, тут поддерживается инвариант, что температура всегда больше абсолютного нуля.

То есть для value objects использование датаклассов — это окей. Потому что value objects это просто неизменяемые простые значения, которые нужны для того, чтобы код был самодокументируемым, чтобы всё было по красоте, чтобы поддерживались инварианты для таких значений, чтобы не путались температуры в Цельсиях и Фаренгейтах и т.п. Примеры value objects это деньги — два объекта, которые хранят 100 рублей, равны друг другу, то есть они сравниваются тоже по значению. Ещё пример — проценты, потому что там значение строго от 0 до 100%, например. Или координаты. Или количество чего-то с единицей измерения, например, три ящика или два килограмма или четыре погонных метра. Или налоговая ставка. Или просто идентификатор, например, `UserId`, `OrderId` и тд.

Ещё датаклассы окей для DTO, Data Transfer Object, то есть структур данных, которые гоняются между слоями приложения. Передаём данные между слоями приложения не кортежами с кучей элементов, не словарями, не типизированными словарями, а именно датаклассами. То есть вот одна функция возвращает пять значений. Как их упаковать? Функция ведь не может вернуть пять значений, функция возвращает в питоне только одно значение, но оно может быть контейнером с пятью другими значениями. Этим контейнером может быть список, кортеж, именованный кортеж, словарь, или вот датакласс. Стоит брать именно датакласс. В редких случаях, когда значений мало, можно использовать кортеж, который будет распаковываться при вызове функции, но чаще всего стоит использовать именно dataclass. Он для этого просто великолепно подходит.

Причём надо стараться почаще использовать в таких случаях для датаклассов тоже параметры `frozen=True` и `slots=True`, потому что для DTO это естественно. Ну и конечно у DTO нет никакой бизнес-логики, это просто контейнер для данных с прописанной структурой и типизацией.

Есть ещё Pydantic и я часто вижу, что люди везде используют его просто в хвост и в гриву, гоняют в DTO для передачи данных между разными слоями системы и тд. Я не считаю это хорошей затеей, чему есть несколько объяснений.

Во-первых, по чистой архитектуре бизнес-логика не должна зависеть от внешних библиотек, а Pydantic — это, конечно, внешняя библиотека; это не часть стандартной библиотеки Python. Уже исходя из этого, бизнес-логика ничего не должна знать о Pydantic.

Во-вторых, как по мне основная задача Pydantic — это удобная работа с JSON, например, валидация входного JSON в вашем веб-сервисе. Для этого Pydantic отлично подходит и здесь он уместен в том же фреймворке FastAPI или где угодно. Также и в обратную сторону, собрать JSON из вложенных сущностей удобно с Pydantic. Но использовать Pydantic где-то ещё я считаю в большинстве случаев неуместным.

Есть ещё внешняя библиотека `attrs`, которую необходимо устанавливать отдельно. На самом деле это в какой-то степени прародитель питоновских датаклассов, часть идей датаклассов взяли как раз из `attrs`. Но функциональных возможностей в `attrs`, конечно, значительно больше, чем в датаклассах. Например, там можно удобно реализовать те самые приватные поля, которые не получается красиво сделать в датаклассах:

```python
import attr


@attr.define
class Something:
    _field1: int = attr.field(alias="field1")
    _field2: int = attr.field(alias="field2")


s = Something(field1=1, field2=2)
print(s)
# Something(_field1=1, _field2=2) 

print(s.field1)  # ошибка
```

Тут незачем объявлять свой метод `__init__` и там вручную присваивать значения полям, всё работает из коробки. Но `__repr__` всё равно отдаёт имена с подчёркиваниями, что нехорошо. Пользователю незачем видеть эти имена с подчёркиваниями. Когда мы написали всё в своём классе, у нас там всё было по красоте с понятным приятным API класса, в том числе и в `__repr__`.

И, как и в случае с pydantic, для attrs актуален вопрос того, что это сторонняя библиотека, которую не стоит использовать в слое бизнес-логики. Чтобы бизнес-логика не зависела от переменчивых внешних библиотек, чтобы обновления и ошибки этих библиотек не влияли на работоспособность самой сложной и самой важной логики проекта — бизнес-логики.

Давайте подытожим.

Ключевая концепция ООП: данные и поведение, связанное с этими данными, должны жить вместе. Если у нас есть отдельно данные, пусть и в виде объекта, и отдельно какие-то функции, которые эти данные обрабатывают, то это плохая реализация ООП. Тогда класс ничего не инкапсулирует, ничего толком не гарантирует, никаких инвариантов не поддерживает, логика обработки данных может быть размазана по всему проекту и тд. Это не ООП. Иногда это уместно, когда класс определяет просто контейнер данных, по сути DTO, в нём нет логики, он чаще всего неизменяем, у него нет жизненного цикла и так далее.

Более того, в правильном ООП класс должен представлять качественную абстракцию, то есть отдавать своим клиентам хороший продуманный минимально необходимый API, через который эти клиенты класса будут с классом взаимодействовать. Всё ненужное для клиентов класс скрывает в приватных и защищённых полях и методах. У класса может быть пара публичных методов, ноль публичных свойств, и с десяток приватных методов — и это прекрасно. Есть понятная продуманная абстракция, которой понятно как пользоваться. Все детали реализации скрыты от клиентов класса. Что уже само по себе хорошо, клиенту класса легко им пользоваться, но также это позволяет впоследствии легко менять детали реализации, то есть приватные поля и методы, сохраняя публичное поведение класса неизменным. Здорово? Здорово.

Вопрос создания качественных абстракций огромный и не такой простой, как может показаться на первый взгляд, возможно мы о нём порассуждаем когда-нибудь отдельно.

Также стоит сказать, что то, о чем я здесь говорю, — это, конечно, не догматизм. Это общая рекомендация, общее правило, если хотите, из которого можно находить исключения. Но чтобы отходить от правила, его хотя бы следует знать и понимать, почему оно именно такое. Надеюсь, что вы теперь это понимаете!

