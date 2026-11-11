# assetout

[English](README.md) | [Русский](README.ru.md)

**Держите большие бинарные файлы вне Git.**  
Храните их в локальном зеркале (`.assetout/`) и синхронизируйте с [GitHub Releases](https://docs.github.com/en/repositories/releasing-projects-on-github).

Простая альтернатива Git LFS, когда нужно только убрать тяжёлые файлы из истории репозитория и раздавать их через Releases.


## Возможности
- Отслеживание больших файлов вне Git (манифест + хеши)
- Локальное зеркало в `.assetout/data/`
- Упаковка в zip-архивы и загрузка в GitHub Releases
- Автоматическое разбиение на несколько архивов при превышении лимита размера
- Опциональная группировка по папке верхнего уровня
- Синхронизация между файлами проекта и зеркалом (`remote` / `local` / `manual`)
- Автоматическое обновление тега релиза, чтобы Source code архивы оставались актуальными
- Обёртки: `git-add`, `git-push`


## Требования
- Python 3.11+
- Git-репозиторий с `origin` на GitHub
- [GitHub CLI (`gh`)](https://cli.github.com/) — авторизованный (`gh auth login`)


## Установка
Рекомендуемый способ (изолированно, как CLI-инструмент):

```bash
pipx install git+https://github.com/ambientphantom/assetout.git
```

Через pip:

```bash
pip install git+https://github.com/ambientphantom/assetout.git
```

Конкретная ветка / тег:

```bash
# ветка
pipx install git+https://github.com/ambientphantom/assetout.git@main

# тег
pipx install git+https://github.com/ambientphantom/assetout.git@v0.2.1
```

Обновление:

```bash
pipx upgrade assetout
# или
pip install --upgrade git+https://github.com/ambientphantom/assetout.git
```

Если команда `assetout` не находится:

```bash
python -m assetout --help
```


## Быстрый старт
```bash
# внутри git-репозитория
assetout init

# зарегистрировать большие файлы (копирует в .assetout/data/, обновляет манифест и .gitignore)
assetout add path/to/big/files_or_dirs

assetout status

# загрузить архивы в GitHub Releases
assetout push

# на другой машине / чистом клоне
assetout pull
assetout sync          # зеркало → файлы проекта
```


## Команды
| Команда     | Описание                                                              |
|-------------|-----------------------------------------------------------------------|
| `init`      | Создаёт `.assetout/` (конфиг, манифест, папка data)                   |
| `add`       | Добавляет большие файлы в зеркало                                     |
| `rm`        | Удаляет из манифеста и зеркала                                        |
| `status`    | Показывает отслеживаемые файлы + статус тега релиза относительно HEAD |
| `push`      | Упаковывает зеркало и загружает в GitHub Releases (по умолчанию обновляет тег) |
| `pull`      | Скачивает архивы релиза в `.assetout/data/`                           |
| `sync`      | Синхронизирует `.assetout/data/` и файлы проекта                      |
| `git-add`   | `assetout add` + `git add`                                            |
| `git-push`  | `git push` + `assetout push`                                          |

### Полезные флаги
```bash
# push
assetout push --replace-all          # удалить старые архивы assetout, затем загрузить
assetout push --update-tag           # переместить тег на HEAD (по умолчанию)
assetout push --no-update-tag        # удалить + пересоздать релиз вместо перемещения тега

# sync / rm
assetout sync --policy local --force
assetout rm path/to/file --project   # также удалить из рабочей директории
```


## Как это работает
```text
файлы проекта  ←→  .assetout/data/  ←→  GitHub Release (zip-архивы)
     ↑ sync              ↑ push / pull
```

- Файлы больше `max_git_file_size` не попадают в Git
- Манифест хранит размер + хеш для каждого пути
- Архивы называются `binaries-01.zip`, `project-binaries-01.zip` и т.д.


## Конфигурация
Создаётся командой `assetout init` в `.assetout/config.yaml`:

```yaml
max_git_file_size: 10MB
hash_algorithm: xxh3
use_only_git_files: true
storage_backend: github_releases
sync_policy: remote          # remote | local | manual

packaging:
  group_by_top_level: false  # true → группировать по имени папки верхнего уровня
  max_archive_size: 1900MB
  max_file_size: 2048MB

github:
  tag: assets-latest
  asset_name: binaries.zip   # устаревшее; имена генерируются автоматически
  owner: ""                  # опционально; берётся из git remote, если пусто
  repo: ""
```


## Типичный рабочий процесс
```bash
# повседневная работа
assetout git-add .
git commit -m "Update assets"
assetout git-push                # = git push + assetout push (с обновлением тега)

# или явно
assetout add assets/
assetout push --replace-all
```

### Обновление тега релиза
По умолчанию `assetout push` перемещает настроенный тег (`assets-latest`) на текущий `HEAD` и делает force-push. Благодаря этому автоматические **Source code** архивы GitHub содержат актуальный код.

```bash
assetout push                  # обновить тег (по умолчанию)
assetout push --no-update-tag  # удалить + пересоздать релиз
```

Используйте `--no-update-tag`, если в команде запрещён force-push тегов.


## Лицензия
MIT