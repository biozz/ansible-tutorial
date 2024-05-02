# Подготовка хостов

У нас будет два Debian хоста в виде Docker-контейнеров. Так как Ansible будет ругаться на пароли при подключении по SSH, мы сразу настраиваем авторизацию по ключу. Файлы конфигурации контейнеров взяты практически один-в-один из репозитория [`Praqma/alpine-sshd`](https://github.com/Praqma/alpine-sshd).

Создать `Dockerfile` внутри `containers`:

```docker
FROM debian:12
COPY entrypoint.sh /
RUN apt-get update \
    && apt-get install -y openssh-server python3.11 \
    && mkdir /var/run/sshd\
    && mkdir -p /root/.ssh \
    && chmod 0700 /root/.ssh \
    && ssh-keygen -A \
    && sed -i s/^#PasswordAuthentication\ yes/PasswordAuthentication\ no/ /etc/ssh/sshd_config \
    && chmod +x entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
CMD ["/usr/sbin/sshd", "-D"]
```

Создать `entrypoint.sh` рядом с `Dockerfile`:

```bash
#!/bin/sh
if [ -z "${AUTHORIZED_KEYS}" ]; then
  echo "AUTHORIZED_KEYS env variable is not set. It is required to setup ssh access to the containers."
  exit 1
fi
echo "Populating /root/.ssh/authorized_keys with the value from AUTHORIZED_KEYS"
echo "${AUTHORIZED_KEYS}" > /root/.ssh/authorized_keys
# Execute the CMD from the Dockerfile:
exec "$@"
```

Создать `docker-compose.yml` внутри `containers`:

```yaml
version: "3"

services:
  deb1:
    build: .
    environment:
      AUTHORIZED_KEYS: ${AUTHORIZED_KEYS}
    ports:
      - "2222:22"
  deb2:
    build: .
    environment:
      AUTHORIZED_KEYS: ${AUTHORIZED_KEYS}
    ports:
      - "2223:22"
```

К этому моменту наша рабочая папка выглядит так:

```
├── .venv
│   └── ...
└── containers
    ├── Dockerfile
    ├── docker-compose.yml
    └── entrypoint.sh
```

Установить переменную окружения для ssh-ключей, собрать образы и запустить:

```bash
export AUTHORIZED_KEYS=$(cat ~/.ssh/id_rsa.pub)
docker compose up --build
```

Если всё ок, то в конце вывода будет так:

```
[+] Running 3/1
 ⠿ Network containers_default   Created
 ⠿ Container containers-deb2-1  Created
 ⠿ Container containers-deb1-1  Created
Attaching to containers-deb1-1, containers-deb2-1
containers-deb2-1  | Populating /root/.ssh/authorized_keys with the value from AUTHORIZED_KEYS env variable ...
containers-deb1-1  | Populating /root/.ssh/authorized_keys with the value from AUTHORIZED_KEYS env variable ...
```

Теперь можно перейти в сосдений терминал и проверить подключение по ssh:

```bash
> ssh root@localhost -p 2222
> ssh root@localhost -p 2223
```

На всякий случай на время обучения можно в `~/.ssh/config` добавить настройку для `localhost`, чтобы не было проблем с добавлением ключей в `known_hosts`:

```
Host localhost
    StrictHostKeyChecking no
```
