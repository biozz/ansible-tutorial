# Определения

**Control node** - любой компьютер, на котором установлен Ansible. С него можно запускать команды и плейбуки.

**Managed nodes** - устройства/серверы в сети, которыми можно управлять с помощью Ansible. Иногда называют "хост". На них Ansible не установлен.

**Inventory** - список, управляемых хостов/нод. Иногда инвентарь называют хостфайлом. Там указывают IP адреса серверов. В инвентаре можно объединять хосты в группы для удобства выкатывания.

**Modules** - единица кода, которую выполняет Ansible. Может настраивать систему, можно объединять в задачи "tasks", можно выполнять в рамках плейбука.

**Tasks** - единица действия, которое может выполнить Ansible. Можно выполнять таски разово с помощью ad-hoc команд.

**Playbooks** - упорядоченный список тасок. Может включать в себя переменные. Пишется на YAML.

