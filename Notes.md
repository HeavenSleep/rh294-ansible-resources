# RH294 - Ansible - Juin 2022

- [RH294 - Ansible - Juin 2022](#rh294---ansible---juin-2022)
  - [Tags](#tags)
  - [Lookup](#lookup)
  - [Variable names](#variable-names)
  - [Prioritization / Lookup hierarchy](#prioritization--lookup-hierarchy)
    - [Files](#files)
    - [Template](#template)
    - [Variables](#variables)

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

## Prioritization / Lookup hierarchy

Ansible will look at very specific places to acces certain resources.

We will summarize them here

### Files

When trying to use files, with `copy` for exemple, it will respect a hierachy to find it stopping at the first match.

```bash
role/files
└── playbook/files
```

### Template

Same as [Files](#files) but for template:

```bash
role/templates
└── playbook/templates
```

### Variables

Variables can be defined and overriden at many levels, here is the hierarchy from the most to least:

1. exta vars (always win precedence)
    ```bash
    ansible-playbook -e "foo=bar"
    ```
2. role and include_role params
   ```yaml
   - name: Pass variables to role
     include_role:
       name: myrole
     vars:
       foo: bar
   ```
   and
   ```yaml
   roles:
     - { name: myrole, foo: bar }
   ```
3. set_facts / registered vars
   ```yaml
   - name: Setting fact
     set_fact:
       foo: bar
   ```
4. include_vars
   ```yaml
   - name: Dynamic include_vars
     include_vars: path/to/foo.yml
   ```
5. task vars (only for the task)
   ```yaml
   - name: Some task
     copy:
       ...
     vars:
       foo: bar
   ```
6. block vars (only for tasks in block)
   ```yaml
   - block:
       - name: Some task in Block
         copy:
           ...
       - ...
     vars:
       foo: bar
   ```
7. role vars (defined in role/vars/main.yml)
8.  play vars_files
    ```yaml
    - hosts: all
      ...
      vars_files:
        - path/to/foo.yml
    ```
9.  play vars_prompt
    ```yaml
    - hosts: all
      ...
      vars_prompt:
        - name: foo
          prompt: What is your foo?
          private: no
    ```
10. play vars
    ```yaml
    - hosts: all
      ...
      vars:
        foo: bar
    ```
11. host facts / cached set_facts
12. playbook host_vars/*
13. inventory host_vars/*
14. inventory file or script host vars
15. playbook group_vars/*
16. inventory group_vars/*
17. playbook group_vars/all
18. inventory group_vars/all
19. inventory file or script group vars
20. role defaults (defined in role/defaults/main.yml)
21. command line values (for example, -u my_user, these are not variables)

*See more*: https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#understanding-variable-precedence