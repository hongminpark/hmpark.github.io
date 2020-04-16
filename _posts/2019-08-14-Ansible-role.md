---
layout: post
title: "[Ansible] Role 을 활용해보자"
author: "Hongmin Park"
tags: ansible
---
[Ansible Role](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html)
변수나 태스크, 핸들러 등의 정의를 구조적으로 할 수 있도록 하는 Ansible의 파일/디렉토리 구조라고 생각하면 된다. 
`ansible-galaxy init [프로젝트명]` 이라고 커맨드를 날리면 해당 프로젝트명으로 디렉토리가 생기고, 디렉토리 안에 아래와 같은 파일들이 생긴다.

```console
[vagrant@controller tomcat85]$ pwd
/home/vagrant/Ansible/tomcat85
[vagrant@controller tomcat85]$ ll
total 4
-rw-rw-r--. 1 vagrant vagrant 1328 Aug 12 07:13 README.md
drwxrwxr-x. 2 vagrant vagrant   22 Aug 12 07:13 defaults
drwxrwxr-x. 2 vagrant vagrant    6 Aug 12 07:13 files
drwxrwxr-x. 2 vagrant vagrant   22 Aug 12 07:13 handlers
drwxrwxr-x. 2 vagrant vagrant   22 Aug 12 07:13 meta
drwxrwxr-x. 2 vagrant vagrant   22 Aug 12 07:13 tasks
drwxrwxr-x. 2 vagrant vagrant    6 Aug 12 07:13 templates
drwxrwxr-x. 2 vagrant vagrant   39 Aug 12 07:13 tests
drwxrwxr-x. 2 vagrant vagrant   22 Aug 12 07:13 vars
```

이 디렉토리에서는 몇가지를 꼭 만족시켜야 한다. <br>
1. 디렉토리 이름 유지
2. 각 디렉토리는 반드시 main.yml을 포함해야 한다.
3. 필요없는 디렉토리는 삭제해도 되지만, 반드시 하나는 있어야한다.
<br>
각 디렉토리는 직관적으로 어떤 역할인지 이해가 가는데 defaults랑 vars는 둘다 변수에 관한 디렉토리이다.<br>
어떻게 다른가 하면 *우선순위*에서 다르고 한다.<br>
Ansible에서 변수는 굉장히 많은 곳에서 정의할 수 있는데.. [(무려 20곳이 넘네.. ?)](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable) 간단히 요약하면 
```
Ansible command에서 넘겨받은 변수
inventory 내의 변수
role: vars
system 변수
role defaults 변수
```
이다.<br>
즉, default보다 vars가 우선순위가더 높다. 
예를들어 tomcat8을 설치할 때 기본적으로 java 1.7을 쓰되, 가끔 java 1.8을 쓰는 경우에 defaults에는 java1.7로, 특정 roles의 vars에만 java1.8로 정의해서 사용할 수 있다. 즉, 동일 변수에 대해서 재정의할 때에 사용한다고 생각하면 된다. 
<br><br>

** role 디렉토리 구조
role하나의 디렉토리는 이러한데, Ansible작업을 구성할 때에 하나의 role을 사용해서 작업을 구성하기보다는 여러 roles들이 합쳐서 작업을 구성한다. roles를 확장하면 아래와 같은 구조가 된다. 
```
site.yml
webservers.yml
fooservers.yml
roles/
   common/
     tasks/
     handlers/
     files/
     templates/
     vars/
     defaults/
     meta/
   webservers/
     tasks/
     defaults/
     meta/
```
이렇게 프로젝트를 구성하게 되면, 보통 site.yml 에서 
```yaml
---
- hosts: webservers
  roles:
    - common
    - webservers
```
위와같은 플레이북을 실행하여 `ansible-playbook site.yml` role들을 불러오게 된다.

