# Хосты

У нас будет два Debian хоста в виде Docker-контейнеров. Так как Ansible будет ругаться на пароли при подключении по SSH, мы сразу настраиваем авторизацию по ключу. Файлы конфигурации контейнеров взяты практически один-в-один из репозитория [`Praqma/alpine-sshd`](https://github.com/Praqma/alpine-sshd).

Создать `Dockerfile`:

```docker
FROM debian:10.4
RUN apt-get update \
    && apt-get install -y openssh-server python3.7 \
    && mkdir /var/run/sshd\
    && mkdir /root/.ssh \
    && chmod 0700 /root/.ssh \
    && ssh-keygen -A \
    && sed -i s/^#PasswordAuthentication\ yes/PasswordAuthentication\ no/ /etc/ssh/sshd_config
COPY docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["/usr/sbin/sshd", "-D"]
```

Создать `docker-entrypoint.sh`:

```yaml
#!/bin/sh
if [ -z "${AUTHORIZED_KEYS}" ]; then
  echo "Need your ssh public key as AUTHORIZED_KEYS env variable. Abnormal exit ..."
  exit 1
fi
echo "Populating /root/.ssh/authorized_keys with the value from AUTHORIZED_KEYS env variable ..."
echo "${AUTHORIZED_KEYS}" > /root/.ssh/authorized_keys
# Execute the CMD from the Dockerfile:
exec "$@"
```

Создать `docker-compose.yml`:

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

Установить переменную окружения для ssh-ключей, собрать образы и запустить:

```bash
export AUTHORIZED_KEYS=$(cat ~/.ssh/id_rsa.pub)
docker-compose up --build
```

Если всё ок, то в конце вывода будет примерно такое:

```bash
Creating containers_deb1_1 ... done
Creating containers_deb2_1 ... done
Attaching to containers_deb2_1, containers_deb1_1
deb2_1  | Populating /root/.ssh/authorized_keys with the value from AUTHORIZED_KEYS env variable ...
deb1_1  | Populating /root/.ssh/authorized_keys with the value from AUTHORIZED_KEYS env variable ...
```

Проверить подключение по ssh:

```bash
> ssh root@localhost -p 2222
> ssh root@localhost -p 2223
```
