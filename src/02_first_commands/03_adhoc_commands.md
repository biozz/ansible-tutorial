# Разовые команды

Также известны как **ad-hoc** команды. Нужны для автоматизации одной задачи на одном или нескольких хостах. Разовые команды легко быстро запустить, но их нельзя переиспользовать. Ansible пишет, что разовые команды нужны скорее для того чтобы показать насколько сильным инструментом он может быть. Хотя концепции разовых команд хорошо транслируются в тему плейбуков.

Самые распространённые примеры использования разовых команд: перезагрузить сервер, скопировать файлы, устанавливать и удалять пакеты, управлять пользователями и т.п.

Разовые команды используют те же принципы, что и плейбуки в том смысле, что они оценивают состояние состояние хоста до выполнения и выполняют действия только если состояние отличается от желаемого.

Самая простая команда:

```bash
> ansible deb1 -a "echo hello"
deb1 | CHANGED | rc=0 >>
hello
```

Тоже самое можно выполнить для группы хостов:

```bash
> ansible debs -m shell -a 'echo $TERM'
deb2 | CHANGED | rc=0 >>
xterm-256color
deb1 | CHANGED | rc=0 >>
xterm-256color
```

Попробуем скопировать файл с нашей машины на удалённые хосты:

```bash
> ansible debs -m copy -a "src=inventory.yml dest=/tmp/foo"
deb2 | CHANGED => {
    "changed": true,
    "checksum": "3b27d49ab53f170b384d5608fb9e3738f69d44fa",
    "dest": "/tmp/foo",
    "gid": 0,
    "group": "root",
    "md5sum": "349210dadef1c2366d62b0c4314c2c5b",
    "mode": "0644",
    "owner": "root",
    "size": 305,
    "src": "/root/.ansible/tmp/ansible-tmp-1590317158.994046-72047-272241656453308/source",
    "state": "file",
    "uid": 0
}
deb1 | CHANGED => {
    "changed": true,
    "checksum": "3b27d49ab53f170b384d5608fb9e3738f69d44fa",
    "dest": "/tmp/foo",
    "gid": 0,
    "group": "root",
    "md5sum": "349210dadef1c2366d62b0c4314c2c5b",
    "mode": "0644",
    "owner": "root",
    "size": 305,
    "src": "/root/.ansible/tmp/ansible-tmp-1590317159.002979-72045-90011462229862/source",
    "state": "file",
    "uid": 0
}
```

Можно убедиться, что файл скопировался:

```bash
> docker exec -it containers_deb1_1 cat /tmp/foo
---
debs:
  hosts:
    deb1:
      ansible_port: 2222
      ansible_host: localhost
      ansible_user: root
      ansible_python_interpreter: /usr/bin/python3.7
    deb2:
      ansible_port: 2223
      ansible_host: localhost
      ansible_user: root
      ansible_python_interpreter: /usr/bin/python3.7
```

Попробуем установить пакет:

```bash
> ansible deb1 -m apt -a "name=acme state=present"
deb1 | CHANGED => {
    "cache_update_time": 1590317476,
    "cache_updated": false,
    "changed": true,
    "stderr": "debconf: delaying package configuration, since apt-utils is not installed\n",
    "stderr_lines": [
        "debconf: delaying package configuration, since apt-utils is not installed"
    ],
    "stdout": "...",
    "stdout_lines": [
			 "..."
    ]
}
```

В документации модуля apt есть описание параметра state. Из описания понятно, что для удаления нужно изменить его на absent. Попробуем "удалить" пакет с хоста, на который мы его не устанавливали:

```bash
> ansible deb2 -m apt -a "name=acme state=absent"
deb2 | SUCCESS => {
    "changed": false
}
```

А теперь тоже самое для всей группы:

```bash
> ansible debs -m apt -a "name=acme state=absent"
deb2 | SUCCESS => {
    "changed": false
}
deb1 | CHANGED => {
    "changed": true,
    "stderr": "",
    "stderr_lines": [],
    "stdout": "...",
    "stdout_lines": [
        "..."
    ]
}
```

Мои первые команды на любом новом сервере обычно такие (ssh под `root`-ом в разных вариациях):

```bash
> apt-get update
> apt-get upgrade
```

Так это будет выглядеть в виде ad-hoc команды:

```bash
> ansible debs -m apt -a "update_cache=yes upgrade=yes"
deb1 | SUCCESS => {
    "changed": false,
    "msg": "...",
    "stderr": "",
    "stderr_lines": [],
    "stdout": "...",
    "stdout_lines": [
        "..."
    ]
}
deb2 | SUCCESS => {
    "changed": false,
    "msg": "...",
    "stderr": "",
    "stderr_lines": [],
    "stdout": "...",
    "stdout_lines": [
        "..."
    ]
}
```

Следующие команды обычно про настройку пользователя и SSH доступа. Я эти команды не могу запомнить, поэтому обычно нахожу статью вроде [Digital Ocean "Initial Server Setup with Debian 10"](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-debian-10) и следую шагам/командам, не читая саму статью. Получается так:

```bash
> adduser biozz
> usermod -aG sudo biozz
```

В наших контейнерах сейчас нет утилиты sudo, можно её установить:

```bash
> ansible debs -m apt -a "name=sudo state=present"
```

Когда создаёшь пользователя есть возможность интерактивно создать пароль. Модуль `[user](https://docs.ansible.com/ansible/latest/modules/user_module.html#user-module)` в Ansible предлагает сделать это с помощью зашифрованной строки:

```bash
ansible debs -m user -a "name=biozz groups=sudo password={{ 'test' | password_hash('sha512') }}"
```

Остался ещё один пример - управление сервисами. Будем испытвать на `nginx`.

```bash
> ansible debs -m apt -a "name=nginx state=present"
```

Запустим сервис

```bash
> ansible debs -m service -a "name=nginx state=started"
deb1 | CHANGED => {
    "changed": true,
    "name": "nginx",
    "state": "started"
}
deb2 | CHANGED => {
    "changed": true,
    "name": "nginx",
    "state": "started"
}
```

Повторное выполнение той же команды, просто чтобы посмотреть на вывод:

```bash
> ansible ansible debs -m service -a "name=nginx state=started"
deb2 | SUCCESS => {
    "changed": false,
    "name": "nginx",
    "state": "started"
}
deb1 | SUCCESS => {
    "changed": false,
    "name": "nginx",
    "state": "started"
}
```

В статье ещё предлагает посмотреть на вывод модуля `setup`, но я считаю, что на этом этапе это не нужно. Там слишком много информации. Чисто ради интереса можно сделать:

```bash
> ansible all -m setup
```
