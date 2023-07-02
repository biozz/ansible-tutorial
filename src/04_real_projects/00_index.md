# Реальные проекты (draft)

- организация
- структура папок
- примеры репозиториев
- мой пример munin (с files)

Предположим есть два сервера:

- serv1 - существующий, хорошо работающий, который уже не надо настраивать
- serv2 - новый сервер, только что купленный, его надо настроить с нуля

```yaml
---
server:
  hosts:
    serv1:
      ansible_port: 22
      ansible_host: 1.2.3.4
      ansible_user: root
      ansible_python_interpreter: /usr/bin/python3.7
    serv2:
      ansible_port: 22
      ansible_host: 5.6.7.8
      ansible_user: root
      ansible_python_interpreter: /usr/bin/python3.7
```

То есть сейчас на serv2 нету ничего, кроме ssh-ключа root-пользователя. И чтобы ansible мог подключиться к этому серверу, нужно сделать:

```yaml
ssh-add ~/.ssh/serv2
```

Теперь нужен playbook для нового сервера:

```yaml
---
- name: Setup new server
  hosts:
    - supreme
  remote_user: root

  roles:
    - common
```

Роль common сейчас содержит только ping, чтобы проверить, что всё работает:

```yaml
---
- name: Ping
  ping:
```

Теперь можно подробно расписать роль:

```yaml

```

- ask-become-pass
- slurp module
- [https://dmsimard.com/2016/03/15/changing-the-ssh-port-with-ansible/](https://dmsimard.com/2016/03/15/changing-the-ssh-port-with-ansible/)
- в ufw ничего не открывать кроме ssh
