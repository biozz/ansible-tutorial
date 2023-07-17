# Использование Taskfile

> **Примечание**: для всех, кто тригернулся на Taskfile, представьте вместо него любой запускатор скриптов или сборщик на ваш вкус

Я фанат [Taskfile](https://taskfile.dev/).
Я даже написал [плагин](https://packagecontrol.io/packages/Taskfile) для Sublime Text.
Я использую Taskfile практическо во всех своих проектах. И Ansible-скрипты не исключение.

Вот небольшой кусочек Taskfile в моем проекте с Ansible-скриптами:

```yaml
version: '3'

tasks:
  requirements:
    cmds:
      - ansible-galaxy install -r requirements.yml --force
    silent: true
  run:
    cmds:
      - ansible-playbook main.yml -i deb1, --tags {{ .CLI_ARGS }}
  run-deb2:
    cmds:
      - ansible-playbook main.yml -i deb2, --tags {{ .CLI_ARGS }}
```

Хочется остановиться на `run` и `run-deb2`.

Чаще всего мы хотим выкатывать какие-то вещи на определенный хост, а запоминать и вводить эту длинную команду ансибла лениво. Поэтому можно написать её один раз и оставить только то, что нужно для кастомизации.

Используются эти команды вот так: `task run -- tag1`, где `tag1` - это тег роли или нескольких ролей.
