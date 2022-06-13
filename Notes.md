# RH294 - Ansible - Juin 2022

- [RH294 - Ansible - Juin 2022](#rh294---ansible---juin-2022)
  - [Tags](#tags)
  - [Lookup](#lookup)
  - [Variable names](#variable-names)

## Tags

A tag can be specified to limit a scope of a playbook run.
It can be defined at the task and role level, such as:

In role tasks:

```yaml
- name: My task
  copy:
    source: ....
    dest: ....
  tags:
    - mytag
    - myothertag

- name: My other task
  copy:
    source: ....
    dest: ....
  tags:
    - myothertag
```

In playbook roles:

```yaml
- hosts: all
  roles:
    - { name: myrole, tags: myroletag }
```

And to run a playbook for specific tags:

```bash
ansible-playbook myplaybook.yml -t mytag,myothertag
```

And to skip specific tags:

```bash
ansible-playbook myplaybook.yml --skip-tags mytag,myothertag
```

*See more at*: https://docs.ansible.com/ansible/latest/user_guide/playbooks_tags.html

## Lookup

The `lookup` function is a powerfull tool that can help you manage large chuck of files without taking a lot of space in your task because you can reference `file`s or `template`s to be dynamically loaded, such as:

```yaml
vars:
  motd_value: "{{ lookup('file', '/etc/motd') }}"
tasks:
  - debug:
      msg: "motd value is {{ motd_value }}"
```

*See more at*: https://docs.ansible.com/ansible/latest/user_guide/playbooks_lookups.html

## Variable names

Variables names are super important because they are made available to the entire scope of the playbook after being defined.

Therefor, it's best to make any variable as specific as you can to avoid conflict.

For exemple, all variables name of a specific role should all be appended by the role name (or a shortened version) such as all variable defined in the `defaults/main.yml` of the role are unique to this role.

If you need to use a more generic variable in your role, use it directly in your templates or tasks without defining it within the role. If it's not defined, an error will occure telling you that it's not defined.
