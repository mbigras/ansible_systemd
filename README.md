README.md

## Setup

```
git clone https://github.com/mbigras/ansible_systemd
cd ansible_systemd
vagrant up
ansible all -m ping
```

## Run with vagrant

Observe how systemctl silently ignores units that don't match the glob pattern:

```
$ vagrant ssh -c 'systemctl stop foo* --user && echo success || echo failure'
success
Connection to 127.0.0.1 closed.
$ vagrant ssh -c 'systemctl stop foo --user && echo success || echo failure'
Failed to stop foo.service: Unit foo.service not loaded.
failure
Connection to 127.0.0.1 closed.
```

## Run with ansible

Observe how systemctl through the shell module silently ignores units that don't match the glob pattern, however, with the systemd module the task fails with error message:

```
Could not find the requested service foo*.service: host
```

```
$ ansible-playbook playbook.yml

PLAY [all] *****************************************************************************

TASK [Gathering Facts] *****************************************************************
ok: [192.168.33.10]

TASK [run systemctl stop foo*.service --user] ******************************************
changed: [192.168.33.10]

TASK [run systemctl stop foo.service --user] *******************************************
fatal: [192.168.33.10]: FAILED! => {"changed": true, "cmd": "systemctl stop foo.service --user", "delta": "0:00:00.003820", "end": "2019-04-17 23:25:33.892407", "msg": "non-zero return code", "rc": 5, "start": "2019-04-17 23:25:33.888587", "stderr": "Failed to stop foo.service: Unit foo.service not loaded.", "stderr_lines": ["Failed to stop foo.service: Unit foo.service not loaded."], "stdout": "", "stdout_lines": []}
...ignoring

TASK [run systemctl stop foo*.service --user] ******************************************
fatal: [192.168.33.10]: FAILED! => {"changed": false, "msg": "Could not find the requested service foo*.service: host"}
...ignoring

TASK [run systemctl stop foo.service --user] *******************************************
fatal: [192.168.33.10]: FAILED! => {"changed": false, "msg": "Could not find the requested service foo.service: host"}
...ignoring

PLAY RECAP *****************************************************************************
192.168.33.10              : ok=5    changed=2    unreachable=0    failed=0

$
$ ls
README.md	ansible.cfg	playbook.retry
Vagrantfile	inventory.ini	playbook.yml
$ ansible-playbook playbook.yml

PLAY [all] *****************************************************************************

TASK [Gathering Facts] *****************************************************************
ok: [192.168.33.10]

TASK [run systemctl stop foo*.service --user] ******************************************
changed: [192.168.33.10]

TASK [run systemctl stop foo.service --user] *******************************************
fatal: [192.168.33.10]: FAILED! => {"changed": true, "cmd": "systemctl stop foo.service --user", "delta": "0:00:00.003811", "end": "2019-04-17 23:33:27.925116", "msg": "non-zero return code", "rc": 5, "start": "2019-04-17 23:33:27.921305", "stderr": "Failed to stop foo.service: Unit foo.service not loaded.", "stderr_lines": ["Failed to stop foo.service: Unit foo.service not loaded."], "stdout": "", "stdout_lines": []}
...ignoring

TASK [run systemctl stop foo*.service --user] ******************************************
fatal: [192.168.33.10]: FAILED! => {"changed": false, "msg": "Could not find the requested service foo*.service: host"}
...ignoring

TASK [run systemctl stop foo.service --user] *******************************************
fatal: [192.168.33.10]: FAILED! => {"changed": false, "msg": "Could not find the requested service foo.service: host"}
...ignoring

PLAY RECAP *****************************************************************************
192.168.33.10              : ok=5    changed=2    unreachable=0    failed=0
```

```
$ cat playbook.yml
---
- hosts: all
  tasks:
    - name: run systemctl stop foo*.service --user
      shell: systemctl stop foo*.service --user

    - name: run systemctl stop foo.service --user
      shell: systemctl stop foo.service --user
      ignore_errors: yes

    - name: run systemctl stop foo*.service --user
      systemd:
        name: foo*.service
        scope: user
        state: stopped
      ignore_errors: yes

    - name: run systemctl stop foo.service --user
      systemd:
        name: foo.service
        scope: user
        state: stopped
      ignore_errors: yes
```