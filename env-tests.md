## Testes do ambiente
Documentação dos testes realizados durante o desenvolvimento.

### Documentação
---

Verificação do arquivo host:
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

Verificação da conexão ansible <--> hosts:
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

Teste de instalação das dependencias do Banco de Dados:
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