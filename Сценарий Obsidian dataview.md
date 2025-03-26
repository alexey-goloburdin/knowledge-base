# Название

**Как назвать видос?**

# Сценарий

>[!question] Начать с этого?
>смотрите, какая крутая штука! Я покажу в этом видео, как её сделать!

Здоров, котаны, хочу поделиться с вами крутейшими штуками, с которыми я тут разобрался в Obsidian. Кто не в курсе, Obsidian это программа для ведения заметок, у меня есть полуторачасовое видео о нём, если пропустили, обязательно посмотрите!

Я там коротенько упоминал плагин dataview, который позволяет создавать базы данных в Obsidian и делать анализ, фильтрацию и визуализацию данных в Obsidian. И сейчас я расскажу вам про dataview подробнее, это оказывается таааакая крутая штука! Там прям можно программировать полноценно на JavaScript, реализуя любую сложную логику. Всё покажу в этом видео.

И также покажу некоторые другие настройки Obsidian, которые я сделал в связи с использованием dataview и не только.

Видео побито по главам, переходите к нужному таймкоду, если не хотите смотреть всё!

# Что показать

10 самых больших заметок:

```dataview
table WITHOUT ID file.name, file.size, link(file.link, "open") as f
sort file.size desc
limit 10
```

Убрать директорию минусом:

```
Table file.ctime as created
From #todevelop and -"008 TEMPLATES"
sort ASCE
Limit 20
```

---

Настройка CSS для дефолтной темы. Настройки Obsidian, Appearance -> CSS Snippets, включить сниппет, который лежит в `.obsidian/snippets/widepage.css`.

# Ещё тулзы

- https://github.com/gfxholo/iconic — иконки для заметок/директорий
- https://github.com/h-sphere/sql-seal — некий аналог dataview, возможно лучше
- https://github.com/SebastianMC/obsidian-custom-sort — сортировка директорий
- https://github.com/epwalsh/obsidian.nvim — nvim plugin for obsidian (?)
- https://github.com/SilentVoid13/Templater

Синхронизация — git плагин и syncthing

# Материалы

- видос Минина — там были интересные плагины
- playlist на ютубе про dataview
- https://habr.com/ru/articles/710356/ — статья про dataview
- https://gennbooth.ru/mysli?entry=1002
- https://readmedium.com/2a275c274936 (см как карточки делает, картинки выводит)
- https://agileadam.com/2022/07/using-dataview-with-charts-in-obsidian/
- https://www.youtube.com/watch?v=G8eOF61wmzI
- https://obsidian.rocks/dataview-in-obsidian-a-beginners-guide/
- https://www.obsidianstats.com/plugins/dataview
- https://www.youtube.com/watch?v=66iB9hNHYQI — куда войти
- https://obsidian.rocks/dataview-in-obsidian-a-beginners-guide/
- http://maxcherepitsa.ru/blog/dataview-obsidian/
- https://cassidoo.co/post/obsidian-dataview/