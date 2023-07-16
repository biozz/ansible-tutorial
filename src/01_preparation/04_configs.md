# Базовый инвентарь

Теперь можно начать настраивать скрипты. Нужно перейти из `containers/` на уровнеь выше и создать `inventory.yml` внутри `ansible/`:

```yaml
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

## Конфигурация Ansible

Создать `ansible.cfg`:

```ini
[defaults]
inventory = inventory.yml
```

Помимо ссылки на инвентарь я обычно устанавливаю эти переменные:

```
deprecation_warnings=False
nocows=True
vault_password_file=/path/to/vault/password/file
```

Подробно про каждую настройку и другие можно почитать здесь [Ansible Configuration Settings](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#ansible-configuration-settings).

## Что должно получиться

```
├── .venv
│   └── ...
├── containers
│   ├── Dockerfile
│   ├── docker-compose.yml
│   └── entrypoint.sh
├── ansible.cfg
└── inventory.yml
```

Теперь мы готовы запускать первые команды. Можно переходить к следующей части.
