# Hello World по-ансибловски

Для проверки подключения к хостам выполнить:

```bash
ansible all -m ping
```

Если всё ок, то вывод будет примерно такой:

```plain
deb2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
deb1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

Возможно потребуется написать `yes` для того чтобы добавить отпечатки контейнеров в `known_hosts` (стандартная процедура для ssh-подключений)
