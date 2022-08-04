# HOMEWORK_BASE_ANSIBLE_8_1_(VITKIN_K_N)

### 1. Подготовка к выполнению.
#### Установите ansible версии 2.10 или выше
```
konstantin@konstantin-forever:~$ ansible --version
ansible 2.10.8
  config file = None
  configured module search path = ['/home/konstantin/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.10.4 (main, Jun 29 2022, 12:14:53) [GCC 11.2.0]

```
#### Создадим свой собственный публичный репозиторий на github с произвольным именем.
```
https://github.com/VitkinKN/BASE_ANSIBLE
```
___
### 2. Выполнение.
#### Попробуем запустить playbook на окружении из test.yml, зафиксируем какое значение имеет факт some_fact для указанного хоста при выполнении playbook'a*
```
konstantin@konstantin-forever:~/DEVOPS_COURSE/HOMEWORKNETOLOGY/BASE_ANSIBLE$ ansible-playbook -i inventory/test.yml site.yml

PLAY [Print os facts] **********************************************************

TASK [Gathering Facts] *********************************************************
[WARNING]: Platform linux on host localhost is using the discovered Python
interpreter at /usr/bin/python3, but future installation of another Python
interpreter could change the meaning of that path. See https://docs.ansible.com
/ansible/2.10/reference_appendices/interpreter_discovery.html for more
information.
ok: [localhost]

TASK [Print OS] ****************************************************************
ok: [localhost] => {
    "msg": "Linux Mint"
}

TASK [Print fact] **************************************************************
ok: [localhost] => {
    "msg": 12
}

PLAY RECAP *********************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```
- *some_fact имеет значение 12*
___
#### Воспользуемся подготовленным (используется docker) или создадим собственное окружение для проведения дальнейших испытаний:*
- *Подготавливаем Docker imades: в контейнер ubuntu устанавливаем дополнительно python3*
```
konstantin@konstantin-forever ~/ansible $ sudo docker pull centos
Using default tag: latest
latest: Pulling from library/centos
a1d0c7532777: Pull complete 
Digest: sha256:a27fd8080b517143cbbbab9dfb7c8571c40d67d534bbdee55bd6c473f432b177
Status: Downloaded newer image for centos:latest

konstantin@konstantin-forever ~/ansible $ sudo docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
405f018f9d1d: Pull complete 
Digest: sha256:b6b83d3c331794420340093eb706a6f152d9c1fa51b262d9bf34594887c2c7ac
Status: Downloaded newer image for ubuntu:latest

konstantin@konstantin-forever:~$ docker exec -it 1427ddb38a33 /bin/bash
root@1427ddb38a33:/# python3 --version
bash: python3: command not found
root@1427ddb38a33:/# apt install python3                     
Reading package lists... Done
...
root@1427ddb38a33:/# python3 --version
Python 3.10.4
```
- *Найдём файл с переменными (group_vars) в котором задаётся найденное в первом пункте значение и поменяем его на 'all default fact'*
```
---examp.yml
  some_fact: "all default fact"
```

```
konstantin@konstantin-forever:~/DEVOPS_COURSE/HOMEWORKNETOLOGY/BASE_ANSIBLE$ ansible-playbook -i inventory/prod.yml site.yml

PLAY [Print os facts] **********************************************************

TASK [Gathering Facts] *********************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] ****************************************************************
ok: [ubuntu] => {
    "msg": "Ubuntu"
}
ok: [centos7] => {
    "msg": "CentOS"
}

TASK [Print fact] **************************************************************
ok: [centos7] => {
    "msg": "el"
}
ok: [ubuntu] => {
    "msg": "deb"
}

PLAY RECAP *********************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```
- *Some_facks: Для Centos имеет значение el. Для Ubuntu имеет значение deb*

#### Добавим факты в group_vars каждой из групп хостов так, чтобы для some_fact получил следующие значения: для deb - 'deb default fact', для el - 'el default fact'
```
---/group_vars/el/examp.yml
  some_fact: "el default fact"
---/group_vars/deb/examp.yml
  some_fact: "deb default fact"
```
```
konstantin@konstantin-forever:~/DEVOPS_COURSE/HOMEWORKNETOLOGY/BASE_ANSIBLE$ ansible-playbook -i inventory/prod.yml site.yml
PLAY [Print os facts] **********************************************************
TASK [Gathering Facts] *********************************************************
ok: [ubuntu]
ok: [centos7]
TASK [Print OS] ****************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}
TASK [Print fact] **************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}
PLAY RECAP *********************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
#### При помощи ansible-vault зашифруйте факты в group_vars/deb и group_vars/el с паролем netology.
```
konstantin@konstantin-forever:~/DEVOPS_COURSE/HOMEWORKNETOLOGY/BASE_ANSIBLE$ ansible-vault encrypt group_vars/deb/examp.yml group_vars/el/examp.yml
New Vault password: 
Confirm New Vault password: 
Encryption successful
```
#### Запустим playbook на окружении prod.yml. При запуске ansible должен запросить пароль. Убедимся в работоспособности.
```
konstantin@konstantin-forever:~/DEVOPS_COURSE/HOMEWORKNETOLOGY/BASE_ANSIBLE$ ansible-playbook -i inventory/prod.yml site.yml --ask-vault-pass
Vault password: 
PLAY [Print os facts] **********************************************************
TASK [Gathering Facts] *********************************************************
ok: [ubuntu]
ok: [centos7]
TASK [Print OS] ****************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}
TASK [Print fact] **************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}
PLAY RECAP *********************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
#### Посмотрим при помощи ansible-doc список плагинов для подключения. Выберим подходящий для работы на control node
```
konstantin@konstantin-forever:~/DEVOPS_COURSE/HOMEWORKNETOLOGY/BASE_ANSIBLE$ ansible-doc --type=connection -l | egrep "execute on controller"
local                          execute on controller
```
- *Требуется добавить local*
```
    file: inventory/prod.yml
    ---
      el:
        hosts:
          centos7:
            ansible_connection: docker
      deb:
        hosts:
          ubuntu:
            ansible_connection: docker
      local:
        hosts:
          localhost:
            ansible_connection: local 
```
#### Запустите playbook на окружении prod.yml. При запуске ansible должен запросить у вас пароль. Убедитесь что факты some_fact для каждого из хостов определены из верных group_vars.
- *Добавил в *
```
konstantin@konstantin-forever:~/DEVOPS_COURSE/HOMEWORKNETOLOGY/BASE_ANSIBLE$ ansible-playbook -i inventory/prod.yml site.yml --ask-vault-pass
Vault password: 
PLAY [Print os facts] **********************************************************
TASK [Gathering Facts] *********************************************************
[WARNING]: Platform linux on host localhost is using the discovered Python
interpreter at /usr/bin/python3, but future installation of another Python
interpreter could change the meaning of that path. See https://docs.ansible.com
/ansible/2.10/reference_appendices/interpreter_discovery.html for more
information.
ok: [localhost]
ok: [ubuntu]
ok: [centos7]
TASK [Print OS] ****************************************************************
ok: [localhost] => {
    "msg": "Linux Mint"
}
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}
TASK [Print fact] **************************************************************
ok: [localhost] => {
    "msg": "all default fact"
}
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}
PLAY RECAP *********************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
___
