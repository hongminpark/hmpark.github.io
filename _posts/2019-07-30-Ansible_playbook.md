---
layout: post
title: "Ansible Playbook 작성하기"
author: "Hongmin Park"
---

지금까지는 커맨드라인으로 ansible 명령어를 통해 핑때리기, 호스트에 작업하기 등을 해보았는데,<br>
이런 방식을 **Ad-hoc** 방식이라고 한다.<br>
하지만, 실제로 명령어가 실어지면 **Ad-hoc** 방식으로는 한계가 있다.<br>
Ansible은 **Ad-hoc** 방식과 **Playbook** 방식으로 호스트를 제어할 수 있는데,<br>
실제로 대부분의 작업은 **Playbook**을 통해 진행된다. **Playbook**은 쉽게말해 **Ad-hoc** 커맨드들을 나열해 놓은 것이라고 생각하면 된다.<br>
**playbook**은 YAML 형태의 파일인데, yaml 파일 작성법은 매우 쉽게 배울 수 있으니 혹시 yaml 을 모른다면 [잠깐 학습하고 오자!](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html)<br>
### playbook.yaml (sample)
``` yaml
---
- hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
  remote_user: root
  tasks:
  - name: ensure apache is at the latest version
    yum:
      name: httpd
      state: latest
  - name: write the apache config file
    template:
      src: /srv/httpd.j2
      dest: /etc/httpd.conf
    notify:
    - restart apache
  - name: ensure apache is running
    service:
      name: httpd
      state: started
  handlers:
    - name: restart apache
      service:
        name: httpd
        state: restarted
```

이제, Playbook을 통해 호스트를 제어하는 방법을 알아보자.<br><br>

### 1. Playbook 작성하기. 
플레이북 파일의 위치는 크게 상관없다. 플레이북 파일의 위치보다는, 인벤토리/변수 파일이 제 위치(ansible.cfg 의 상대경로)에 있는 것이 더 중요하다.<br>
##### playbook_ex.yaml
```yaml
---
- hosts: host1 # play1
  tasks:
  - ping:
      data: abc
- hosts: host2 # play2
  tasks:
  - ping:
      data: def
```
위와같이 playbook은 여러 play들로 구성되어있고, 각 play는 hosts, task 등의 요소들로 구성되어있다.<br>
하나의 playbook에 여러 play를 둘 수도 있고, play마다 playbook을 따로 만들 수도 있다.<br>

### 2. Playbook 실행하기.
기본적으로 `ansible-playbook [플레이북 파일명]`로 플레이북을 실행할 수 있다.
```console
[vagrant@controller tmp]$ ansible-playbook ping.yaml

PLAY [mgmt] ***************************************************************************

TASK [Gathering Facts] ****************************************************************
ok: [host2]
ok: [host1]

TASK [ping] ***************************************************************************
ok: [host1]
ok: [host2]

PLAY RECAP ****************************************************************************
host1                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
host2                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

플레이북의 문법확인이 필요한 경우 `ansible-playbook [플레이북 파일명] --syntax-check`로 문법체크를 수시로 하자.
```console
[vagrant@controller tmp]$ ansible-playbook ping.yaml --syntax-check
ERROR! Syntax Error while loading YAML.
  did not find expected '-' indicator

The error appears to be in '/tmp/ping.yaml': line 3, column 3, but may
be elsewhere in the file depending on the exact syntax problem.

The offending line appears to be:

-  hosts: mgmt
  tasks: # module name
  ^ here

```



