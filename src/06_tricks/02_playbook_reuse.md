# Переиспользование плейбука

Иногда приходишь к тому, что плейбук становится универсальным или однотипным для каких-то нужд и при этом ты постоянно его копируешь. Копировать в целом неплохо, но хочется сократить количество открытых файлов и вносимых изменений.

Суть этого трюка в двух моментах:

- вынести общие таски деплоя в отдельный файл, чтобы импортировать
- кастомизировать отдельные таски деплоя в переменных роли

Вот древо файлов для этого трюка:

```
├── app
│   ├── defaults
│   │   └── main.yml
│   └── tasks
│       └── main.yml
└── common
    └── app.yml
```

Как обычно, исходники можно найти тут - [`src/00_sources/06_tricks/02_playbook_reuse`](https://github.com/biozz/ansible-tutorial/tree/master/src/00_sources/06_tricks/02_playbook_reuse).

Предположим есть такой плейбук:

```yaml
# 01_example/common/app.yml
{{#include ../00_sources/06_tricks/02_playbook_reuse/01_example/common/app.yml}}
```

Который можно кастомизировать с помощью переменных вот так:

```yaml
{{#include ../00_sources/06_tricks/02_playbook_reuse/01_example/app/defaults/main.yml}}
```

Из интересных вещей тут следующие:

- команды для запуска внутри контейнера в одной переменной в виде списка

```yaml
{{#include ../00_sources/06_tricks/02_playbook_reuse/01_example/app/defaults/main.yml:11:14}}
```

- лейблы для контейнера разложены на `key`/`value`, которые дальше в плейбуке раскладываются с помощью `items2dict`

```yaml
# 01_example/app/defaults/main.yml
{{#include ../00_sources/06_tricks/02_playbook_reuse/01_example/app/defaults/main.yml:16:28}}
```

```yaml
# 01_example/common/app.yml
{{#include ../00_sources/06_tricks/02_playbook_reuse/01_example/common/app.yml:47}}
```

И теперь всё, что остаётся для каждого нового приложения - это заимпортировать из `common/app.yml` и изменить переменные по вкусу:

```yaml
# 01_example/app/tasks/main.yml
{{#include ../00_sources/06_tricks/02_playbook_reuse/01_example/app/tasks/main.yml}}
```
