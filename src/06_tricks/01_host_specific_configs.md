# Разные конфиги для разных хостов

> **TL;DR**: в названии файлов конфигов можно указывать `inventory_hostname_short`, например `telegraf.myhost.conf`, `_short` чтобы можно было в инвентаре делать `myhost.this` и `myhost.that`, так как это удобно для разделения ssh-конфигов

Иногда бывает так, что нужно запускать один и тот же плейбук или одну и ту же роль для разных хостов, но чтобы у них были разные конфиги. Кастомизация содержимого конфига будет сильно зависеть от целей. Иногда достаточно прокинуть переменную из инвентаря или указать какой-нибудь флаг в `defaults` и вручную менять его перед запуском.

Нас интересует полностью автоматический метод, поэтому остановимся на переменных инвентаря. Примеры написаны для [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/), этакий комбайн для сбора, обработки и отправки метрик.

## Переменные инвентаря

Полностью рабочий пример проекта можно найти тут: [`src/00_sources/06_tricks/01_host_specific_configs/01_inventory_vars`](https://github.com/biozz/ansible-tutorial/tree/master/src/00_sources/06_tricks/01_host_specific_configs/01_inventory_vars). Запускается из папки с инвентарём с помощью `ansible-playbook main.yml`.

Вот все его файлы:

```
├── inventory
├── main.yml
└── roles
    └── telegraf
        ├── defaults
        │   └── main.yml
        ├── tasks
        │   └── main.yml
        └── templates
            └── telegraf.conf.j2
```

Сначала рассмотрим инвентарь:

```ini
{{#include ../00_sources/06_tricks/01_host_specific_configs/01_inventory_vars/inventory}}
```

`is_net_enabled=true` - это тот самый флаг, который поможет внутри шаблона конфига. Условно, будем считать что с одного хоста мы хотим собирать метрики сети (`net`), а с другого нет.

> **Примечание**: переменные инвентаря в таком виде будут в [первой половине](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#understanding-variable-precedence) иерархии переменных и велика вероятность, что они перезапишутся из других мест, если указать где-то еще. Будьте внимательны.

И вот он в `telegraf.conf.j2`, шаблоне, который рендерится в `tasks/main.yml`:

```toml
{{#include ../00_sources/06_tricks/01_host_specific_configs/01_inventory_vars/roles/telegraf/templates/telegraf.conf.j2:condition}}
```

Проблема в том, что со временем конфиги могут сильно отличаться и if-чики в коде начнут сбивать с толку.

## `inventory_hostname_short`

То есть нам по сути нужно как-то завязаться на переменную названия хоста.

Для этого сценария дерево файлов изменится лишь в `templates/`:

```
└── templates
    ├── telegraf.deb1.conf.j2
    └── telegraf.deb2.conf.j2
```

Возможно появился вопрос, почему именно `_short`, а не обычный `inventory_hostname`? Это небольшой хак, который позволяет указывать один и тот же хост, только с разными суффиксами. Например, это полезно, когда подключаешься к домашнему серваку дома через локальный IP, а когда на выезде - через какой-нибудь VPN. Так вот, `_short`, как уже догадались, возьмёт часть до точки.

```ini
{{#include ../00_sources/06_tricks/01_host_specific_configs/02_hostname_short/inventory}}
```

> Интересный факт, в [разделе Special variables](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html#term-inventory_hostname_short) в официальной доке Ansible про это не написано. Зато наглядно показано в [этой статье](http://www.freekb.net/Article?id=2428).

И тогда в `tasks/main.yml` можно написать так:

```yaml
{{#include ../00_sources/06_tricks/01_host_specific_configs/02_hostname_short/roles/telegraf/tasks/main.yml:hostname}}
```

На выходе получилось два независимых конфига на двух разных хостах.

Это не супер-масштабируемо, так как при добавлении нового хоста для него нужно будет добавлять ещё один конфиг. Зато явно и наглядно.
