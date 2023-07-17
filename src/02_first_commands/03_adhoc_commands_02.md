# Разовые команды - 2

Следующие часто выполняемые команды обычно про настройку пользователя и SSH доступа. Я эти команды не могу запомнить, поэтому обычно нахожу статью вроде [Digital Ocean "Initial Server Setup with Debian 10"](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-debian-10) и следую шагам/командам, не читая саму статью. Получается так:

```bash
> adduser biozz
> usermod -aG sudo biozz
```

В наших контейнерах сейчас нет утилиты `sudo`, можно её установить:

```bash
> ansible debs -m apt -a "name=sudo state=present"
```

Когда создаёшь пользователя есть возможность интерактивно создать пароль. Модуль [`user`](https://docs.ansible.com/ansible/latest/modules/user_module.html#user-module) в Ansible предлагает сделать это с помощью зашифрованной строки:

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
