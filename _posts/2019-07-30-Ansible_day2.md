---
layout: post
title: "Ansible 사용하기[Day2]"
author: "Hongmin Park"
---

### Ansible 설정파일 - ansible.cfg
Ansible의 설정파일은 ansible.cfg 인데, 다양한 곳에 위치할 수 있다. <br>
아래와 같은 순서가 상위로 먹힌다.<br>
{% highlight markdown %}
ANSIBLE_CONFIG (유저의 환경변수로 export)
ansible.cfg (현재 디렉토리의)
.ansible.cfg (홈 디렉토리의)
/etc/ansible/ansible.cfg (default)
{% endhighlight %}
많은 곳에서 설정 가능한데, 헷갈릴 수 있으니 **ANSIBLE_CONFIG**로 설정하는 것이 best라고 생각한다. <br><br>

현재 ansible.cfg의 설정을 확인하려면 `ansible-config view` 로 확인할 수 있다. 
{% highlight console %}
[vagrant@controller ~]$ ansible-config view
[defaults]
inventory = hosts

[privilege_escalation]
become = false
{% endhighlight %}

또한, 가능한 모든 설정을 보려면 `ansible-config list config`로 볼 수 있다.
{% highlight console%}
[vagrant@controller ~]$ ansible-config list config
ACTION_WARNINGS:
  default: true
  description: [By default Ansible will issue a warning when received from a task
      action (module or action plugin), These warnings can be silenced by adjusting
      this setting to False.]
  env:
  - {name: ANSIBLE_ACTION_WARNINGS}
  ini:
  - {key: action_warnings, section: defaults}
  name: Toggle action warnings
  type: boolean
  version_added: '2.5'
AGNOSTIC_BECOME_PROMPT:
  default: true
  description: Display an agnostic become prompt instead of displaying a prompt contain
    the command line supplied become method
  env:
  - {name: ANSIBLE_AGNOSTIC_BECOME_PROMPT}
  ini:
  - {key: agnostic_become_prompt, section: privilege_escalation}
  name: Display an agnostic become prompt
  type: boolean
  version_added: '2.5'
  yaml: {key: privilege_escalation.agnostic_become_prompt}
ALLOW_WORLD_READABLE_TMPFILES:
  default: false
  ...
{% endhighlight %}

`ansible-config dump`을 이용하면, default를 포함하여 현재 적용되어있는 모든 설정을 확인할 수 있다.
{% highlight console %}
[vagrant@controller ~]$ ansible-config dump
ACTION_WARNINGS(default) = True
AGNOSTIC_BECOME_PROMPT(default) = True
ALLOW_WORLD_READABLE_TMPFILES(default) = False
ANSIBLE_CONNECTION_PATH(default) = None
ANSIBLE_COW_PATH(default) = None
ANSIBLE_COW_SELECTION(default) = default
ANSIBLE_COW_WHITELIST(default) = ['bud-frogs', 'bunny', 'cheese', 'daemon', 'default',
ANSIBLE_FORCE_COLOR(default) = False
ANSIBLE_NOCOLOR(default) = False
ANSIBLE_NOCOWS(default) = False
ANSIBLE_PIPELINING(default) = False
ANSIBLE_SSH_ARGS(default) = -C -o ControlMaster=auto -o ControlPersist=60s
ANSIBLE_SSH_CONTROL_PATH(default) = None
\<span style="color:blue"\>DEFAULT_BECOME(/home/vagrant/.ansible.cfg) = False\</span\>
{% endhighlight %}
이 때, 사용자의 cfg파일에 의해 적용된 값은 노란색으로 다르게 표시되는 것을 확인할 수 있다.
