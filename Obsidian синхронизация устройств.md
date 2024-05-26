Текущая схема:

- каждый компьютер соединен с сервером через Syncthing. Все синхронизации между компьютерами идут через этот сервер автоматически
- телефон Huawei Mate XS не удалось подружить должным образом с Syncthing, синхронизация происходит через git, причём запускаемый только в Termux, не через Git-плагин Obsidian, потому что он плохо работает на телефоне (в отличие от компьютера)
- текущая директория также бэкапится [на GitHub](https://github.com/alexey-goloburdin/knowledge-base/) через [плагин Git для Obsidian](https://github.com/denolehov/obsidian-git) по `CTRL`+`P`
- на сервере и компьютерах директория `Sync` настроена с File Versioning: Simple File Versioning. Параметры:
	- Clear out after: 60 days
	- Keep Versions: 5
	- Cleanup Interval: 3600 seconds (default value)
- на сервере в настройках директории стоит Ignore patterns:

```
.obsidian
.git
.DS_Store
```
