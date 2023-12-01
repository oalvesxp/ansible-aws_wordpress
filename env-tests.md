## Testes do ambiente
Documentação dos testes realizados durante o desenvolvimento.

### Documentação
---

1. Verificação do arquivo host:
```
$ ansible-inventory --list
{
    "_meta": {
        "hostvars": {
            "mysql": {
                "ansible_host": "3.85.230.105",
                "ansible_ssh_private_key_file": "./ansible.pem"
            },
            "webapp": {
                "ansible_host": "3.85.230.105",
                "ansible_ssh_private_key_file": "./ansible.pem"
            }
        }
    },
    "all": {
        "children": [
            "ungrouped",
            "prod"
        ]
    },
    "database": {
        "hosts": [
            "mysql"
        ]
    },
    "prod": {
        "children": [
            "wordpress",
            "database"
        ]
    },
    "wordpress": {
        "hosts": [
            "webapp"
        ]
    }
}

```

2. Verificação da conexão ansible <--> hosts:
```
$ ansible prod -m ping -uadmin 
mysql | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
webapp | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

3. Teste de instalação das dependencias do Banco de Dados:
```
$ ansible-playbook playbook-wordpress-startconfig.yml -i inventory.ini -uadmin -C

PLAY [database] *****************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************
ok: [mysql]

TASK [Instalando o MySQL] *******************************************************************************************************************
changed: [mysql] => (item=mariadb-server)
changed: [mysql] => (item=python3-mysqldb)

PLAY RECAP **********************************************************************************************************************************
mysql                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

4. Teste de instalação do php8.2 nos hosts wordpress:
```
$ ansible-playbook playbook-wordpress-startconfig.yml -t php -i inventory.ini -uadmin -C

PLAY [database] *****************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************
ok: [mysql]

PLAY [wordpress] ****************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************
ok: [webapp]

TASK [Instalando o PHP] *********************************************************************************************************************
changed: [webapp] => (item=php8.2)
changed: [webapp] => (item=php8.2-gd)
changed: [webapp] => (item=php8.2-mcrypt)
changed: [webapp] => (item=php8.2-mysql)

PLAY RECAP **********************************************************************************************************************************
mysql                      : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
webapp                     : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

5. Teste de instalação do apache2 nos hosts wordpress:
```
$ ansible-playbook playbook-wordpress-startconfig.yml -t apache -i inventory.ini -uadmin -C

PLAY [database] *****************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************
ok: [mysql]

PLAY [wordpress] ****************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************
ok: [webapp]

TASK [Instalando o Apache] ******************************************************************************************************************
changed: [webapp] => (item=apache2)
changed: [webapp] => (item=libapache2-mod-php8.2)

PLAY RECAP **********************************************************************************************************************************
mysql                      : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
webapp                     : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

---

Agora comecei a criar uma estrutura mais segmentada do ansible para organizar todas as roles e tasks.

6. Testando o novo playbook-configstart-wp
```
$ ansible-playbook playbook-configstart-wp.yml -i inventory.ini -uadmin -C

PLAY [prod] *********************************************************************************************************************************

TASK [Instalando o Python3] *****************************************************************************************************************
skipping: [mysql]
skipping: [webapp]

PLAY RECAP **********************************************************************************************************************************
mysql                      : ok=0    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
webapp                     : ok=0    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
```

7. Testando a nova estrutura para o server:
```
$ ansible-playbook playbook-configstart-wp.yml -i inventory.ini -uadmin -C

PLAY [prod] *********************************************************************************************************************************

TASK [Atualizando o apt-cache] **************************************************************************************************************
changed: [mysql]
changed: [webapp]

TASK [Instalando o Python3] *****************************************************************************************************************
skipping: [webapp]
skipping: [mysql]

PLAY [wordpress] ****************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************
ok: [webapp]

TASK [server : Instalando o servidor Web] ***************************************************************************************************
changed: [webapp] => (item=apache2)
changed: [webapp] => (item=libapache2-mod-php8.2)

PLAY RECAP **********************************************************************************************************************************
mysql                      : ok=1    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
webapp                     : ok=3    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
```

8. Testando as tasks de instalação do php8.2:
```
$ ansible-playbook playbook-configstart-wp.yml -t install-php -i inventory.ini -uadmin -C

PLAY [prod] *********************************************************************************************************************************

PLAY [wordpress] ****************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************
ok: [webapp]

TASK [php : Instalando as dependências do php.8.2] ******************************************************************************************
changed: [webapp] => (item=php8.2)
changed: [webapp] => (item=php8.2-gd)
changed: [webapp] => (item=php8.2-mcrypt)
changed: [webapp] => (item=php8.2-mysql)

PLAY RECAP **********************************************************************************************************************************
webapp                     : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```