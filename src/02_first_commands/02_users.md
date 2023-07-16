# Пользователи

В нашей конфигурации мы используем `root`, но это не всегда так, поэтому в доках описан такой сценарий:

```bash
# as bruce
> ansible all -m ping -u bruce
# as bruce, sudoing to root (sudo is default method)
> ansible all -m ping -u bruce --become
# as bruce, sudoing to batman
> ansible all -m ping -u bruce --become --become-user batman
```
