# Установка и папки

## Необходимое

Нам потребуется несколько утилит, чтобы следовать описанию:

- [Docker](https://docs.docker.com/engine/install/) - автоматизировать мы будем контейнеры, так что нужен инструмент для контейниризации. Конкретно Docker не обязателен, можно использовать любой доступный инструмент, который понимает формат `Dockerfile`
- [Python](https://www.python.org/downloads/) - основной способ установки Ansible это pip-пакет

## Папки

- создать папку `ansible`, в ней будут находиться все конфиги и отправная точка для команд
- создать папку `containers` в папке `ansible`

> Примечание: Название папки не принципиально

## Ansible

Официальный гайд будто намекает, что можно использовать виртуальную среду. Давайте создадим её и активируем.

> [Use pip in your selected Python environment to install the Ansible package](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#pip-install)

```bash
python -m venv .venv
source .venv/bin/activate
```

А дальше сам `ansible`:

```bash
pip install ansible
```

