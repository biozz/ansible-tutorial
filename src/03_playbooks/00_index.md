# Плейбуки

Плейбуки совсем не похожи на ad-hoc команды из прошлой части. Если модули Ansible это инструменты, то плейбуки - это инструкции по их использованию. Они подходят для выкатывания сложных конфигураций и оркестрации хостов. В отличие от разовых команд, плейбуки принято хранить в системе контроля версий.

## Синтаксис

Плейбуки пишут на YAML. В мануале предлагается установить и использовать утилиты для валидации, типа [ansible-lint](https://github.com/ansible/ansible-lint), но для начала хватит простой подстветки. Отличный источник для вдохновения и примеров - [ansible-examples](https://github.com/ansible/ansible-examples).

Плейбук состоит из набора шагов, которые должны привести хост к состоянию его конечному состоянию, то есть к его роли. В мануале пишут про отсылки к спортивным играм, отсюда и термины. В каждом шаге может быть несколько задач.

Если взять часть команд из 2-ой статьи и собрать их, то получится:

```yaml
---
- hosts: debs
  tasks:
    - name: check terminal setup
      shell:
        cmd: echo $TERM
    - name: install acme package
      apt:
        name: acme
        state: present
        update_cache: true
    - name: install sudo package
      apt:
        name: sudo
        state: present
    - name: setup biozz user
      user:
        name: biozz
        groups: sudo
        password: "{{ 'test' | password_hash('sha512') }}"
```

Сохраним это в `playbook.yml` и запустим (убрал повторяющийся вывод):

```bash
ansible-playbook playbook.yml
PLAY [debs] ********************************************************************

TASK [Gathering Facts] *********************************************************
ok: [deb1]
ok: [deb2]

TASK [xxx] ****************************************************
changed: [deb1]
changed: [deb2]

PLAY RECAP *********************************************************************
deb1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
deb2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Если в последней команде оставить без двойных кавычек, то получим ошибку. Что сразу же говорит о том, что есть шаблонные выражения и надо не забывать их экранировать кавычками.

```bash
ERROR! We were unable to read either as JSON nor YAML, these are the errors we got from each:
JSON: Expecting value: line 1 column 1 (char 0)

Syntax Error while loading YAML.
  found character that cannot start any token

The error appears to be in '/Users/biozz/other/ansible/playbook.yml': line 20, column 29, but may
be elsewhere in the file depending on the exact syntax problem.

The offending line appears to be:

        groups: sudo
        password: {{ 'test' | password_hash('sha512') }}
                            ^ here
We could be wrong, but this one looks like it might be an issue with
missing quotes. Always quote template expression brackets when they
start a value. For instance:

    with_items:
      - {{ foo }}

Should be written as:

    with_items:
      - "{{ foo }}"
```

Мы уже довольно много команд выполнили под root-ом, что считается небезопасным подходом. А у нас уже есть пользователь, поэтому можно переключиться на него для выполнения команд:

```yaml
---
- hosts: debs
	remote_user: biozz
	become: yes
	become_method: sudo
```

[https://docs.ansible.com/ansible/latest/user_guide/playbooks.html#working-with-playbooks](https://docs.ansible.com/ansible/latest/user_guide/playbooks.html#working-with-playbooks)

[https://github.com/ansible/ansible-examples/tree/master/](https://github.com/ansible/ansible-examples/tree/master/)

[https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html#handlers-running-operations-on-change](https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html#handlers-running-operations-on-change)

## Материалы из этой части

- [Ansible Playbooks](https://docs.ansible.com/ansible/latest/user_guide/playbooks.html)
- [Ansible Playbook Examples](https://github.com/ansible/ansible-examples)
- [ansible-lint](https://github.com/ansible/ansible-lint)
- [Ansible Module `shell`](https://docs.ansible.com/ansible/latest/modules/shell_module.html)
