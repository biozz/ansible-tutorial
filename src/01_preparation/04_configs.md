# Базовый инвентарь

Создать `inventory.yml`:

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

```bash
[defaults]
inventory = /path/to/inventory.yml
```

## Что должно получиться

```bash
├── containers
│   ├── Dockerfile
│   ├── docker-compose.yml
│   └── docker-entrypoint.sh
├── ansible.cfg
└── inventory.yml
```
