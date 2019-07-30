---
layout: post
title: "Ansible Playbook 작성하기"
author: "Hongmin Park"
---

지금까지는 커맨드라인으로 ansible 명령어를 통해 핑때리기, 호스트에 작업하기 등을 해보았는데,<br>
이런 방식을 **Ad-hoc** 방식이라고 한다.<br>
하지만, 실제로 명령어가 실어지면 **Ad-hoc** 방식으로는 한계가 있다.<br>
Ansible은 **Ad-hoc** 방식과 **Playbook** 방식으로 호스트를 제어할 수 있는데,<br>
실제로 대부분의 작업은 Playbook을 통해 진행된다. Playbook은 쉽게말해 Ad-hoc 커맨드들을 나열해 놓은 것이라고 생각하면 된다.<br>
playbook은 YAML 형태의 파일인데, yaml 파일 작성법은 매우 쉽게 배울 수 있으니 혹시 yaml 을 모른다면 [잠깐 학습하고 오자!](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html)<br>
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


