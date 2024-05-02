# Отладка

- разобрать пример такого конфига

```ini
[defaults]
ansible_python_interpreter = /usr/bin/python3
filter_plugins = ./filter_plugins
host_key_checking = False
forks = 20
roles_path = ./roles
library = ./library
stdout_callback = yaml
bin_ansible_callbacks = True
deprecation_warnings = False
callbacks_enabled = profile_tasks
check_mode_markers = true
display_skipped_hosts = false
show_custom_stats = true ; ← вот это вроде статы по времени тасок

[ssh_connection]
control_path = %(directory)s/%%h-%%p-%%r
pipelining = True
ssh_args = -o ForwardAgent=yes -o ControlMaster=auto -o ControlPersist=60s
```

- про `--check` и `--diff`

- про молекулу
