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

9. Teste de instalação das dependências do MySQL:
```
$ ansible-playbook playbook-configstart-wp.yml -t setup-mysql -i inventory.ini -uadmin -C

PLAY [prod] *********************************************************************************************************************************

PLAY [wordpress] ****************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************
ok: [webapp]

PLAY [database] *****************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************
ok: [mysql]

TASK [mysql : Instalando as dependências do MySQL] ******************************************************************************************
changed: [mysql] => (item=mariadb-server)
changed: [mysql] => (item=python3-mysqldb)

PLAY RECAP **********************************************************************************************************************************
mysql                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
webapp                     : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

10. Testando a criação do banco de dados:
```
$ ansible-playbook playbook-configstart-wp.yml -t setup-mysql,setup-db -i inventory.ini -uadmin

PLAY [prod] **************************************************************************************************************************************************

PLAY [wordpress] *********************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [webapp]

PLAY [database] **********************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [mysql]

TASK [mysql : Instalando as dependências do MySQL] ***********************************************************************************************************
changed: [mysql] => (item=mariadb-server)
changed: [mysql] => (item=python3-mysqldb)

TASK [mysql : Criando o Banco de Dados] **********************************************************************************************************************
changed: [mysql]

PLAY RECAP ***************************************************************************************************************************************************
mysql                      : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
webapp                     : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
``` 

11. Testando a task de criação de usuário do banco:
```
$ ansible-playbook playbook-configstart-wp.yml -t setup-dbuser -i inventory.ini -uadmin

PLAY [prod] **************************************************************************************************************************************************

PLAY [wordpress] *********************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [webapp]

PLAY [database] **********************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [mysql]

TASK [mysql : Criando o usuário do MySQL] ********************************************************************************************************************
changed: [mysql]

PLAY RECAP ***************************************************************************************************************************************************
mysql                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
webapp                     : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

12. Testando o usuário criado: 
```
admin@ip-172-31-37-215:~$ sudo mysql -ustaging_user -p 
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 34
Server version: 10.11.4-MariaDB-1~deb12u1 Debian 12

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| staging_wp         |
+--------------------+
2 rows in set (0.000 sec)
```

13. Teste de download do WordPress:
```
$ ansible-playbook playbook-configstart-wp.yml -t get-wp -i inventory.ini -uadmin

PLAY [prod] **************************************************************************************************************************************************

PLAY [wordpress] *********************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [webapp]

TASK [wordpress : Baixando o WordPress] **********************************************************************************************************************
changed: [webapp]

PLAY [database] **********************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [mysql]

PLAY RECAP ***************************************************************************************************************************************************
mysql                      : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
webapp                     : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

14. Verificando o arquivo do WordPress no diretório:
```
admin@ip-172-31-37-215:~$ ls -l /tmp/latest.tar.gz 
-rw-r--r-- 1 admin admin 24479162 Dec  1 18:25 /tmp/latest.tar.gz
```

15. Teste da estrutura ansible atá aqui:
```
PLAY RECAP ***************************************************************************************************************************************************
mysql                      : ok=7    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
webapp                     : ok=7    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

Arquivos do WordPress:
```
admin@ip-172-31-37-215:~$ ls -l /var/www/
total 8
drwxr-xr-x 2 root   root    4096 Dec  1 18:40 html
drwxr-xr-x 5 nobody nogroup 4096 Nov  9 00:45 wordpress
admin@ip-172-31-37-215:~$ ls -l /var/www/wordpress/
total 228
-rw-r--r--  1 nobody nogroup   405 Feb  6  2020 index.php
-rw-r--r--  1 nobody nogroup 19915 Jan  1  2023 license.txt
-rw-r--r--  1 nobody nogroup  7399 Jul  5 17:41 readme.html
-rw-r--r--  1 nobody nogroup  7211 May 12  2023 wp-activate.php
drwxr-xr-x  9 nobody nogroup  4096 Nov  9 00:45 wp-admin
-rw-r--r--  1 nobody nogroup   351 Feb  6  2020 wp-blog-header.php
-rw-r--r--  1 nobody nogroup  2323 Jun 14 14:11 wp-comments-post.php
-rw-r--r--  1 nobody nogroup  3013 Feb 23  2023 wp-config-sample.php
drwxr-xr-x  4 nobody nogroup  4096 Nov  9 00:45 wp-content
-rw-r--r--  1 nobody nogroup  5638 May 30  2023 wp-cron.php
drwxr-xr-x 27 nobody nogroup 12288 Nov  9 00:45 wp-includes
-rw-r--r--  1 nobody nogroup  2502 Nov 26  2022 wp-links-opml.php
-rw-r--r--  1 nobody nogroup  3927 Jul 16 12:16 wp-load.php
-rw-r--r--  1 nobody nogroup 50924 Sep 29 22:01 wp-login.php
-rw-r--r--  1 nobody nogroup  8525 Sep 16 06:50 wp-mail.php
-rw-r--r--  1 nobody nogroup 26409 Oct 10 14:05 wp-settings.php
-rw-r--r--  1 nobody nogroup 34385 Jun 19 18:27 wp-signup.php
-rw-r--r--  1 nobody nogroup  4885 Jun 22 14:36 wp-trackback.php
-rw-r--r--  1 nobody nogroup  3154 Sep 30 07:39 xmlrpc.php
```

16. Teste de alteração o vhost apache:
```
$ ansible-playbook playbook-configstart-wp.yml -t vhost -i inventory.ini -uadmin

PLAY [prod] **************************************************************************************************************************************************

PLAY [wordpress] *********************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [webapp]

TASK [webserver : Atualizando o vHost] ***********************************************************************************************************************
changed: [webapp]

PLAY [database] **********************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [mysql]

PLAY RECAP ***************************************************************************************************************************************************
mysql                      : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
webapp                     : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

17. Testando a task de copia do config.php
```
$ ansible-playbook playbook-configstart-wp.yml -t copy-php-config -i inventory.ini -uadmin

PLAY [prod] **************************************************************************************************************************************************

PLAY [wordpress] *********************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [webapp]

TASK [wordpress : Copiando o arquivo template do config.php] *************************************************************************************************
changed: [webapp]

PLAY [database] **********************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [mysql]

PLAY RECAP ***************************************************************************************************************************************************
mysql                      : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
webapp                     : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

18. Teste da task de edição do wp-config.php
```
$ ansible-playbook playbook-configstart-wp.yml -t config-wp -i inventory.ini -uadmin

PLAY [prod] **************************************************************************************************************************************************

PLAY [wordpress] *********************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [webapp]

TASK [wordpress : Update WordPress config file] **************************************************************************************************************
changed: [webapp] => (item={'regex': 'database_name_here', 'value': 'staging_wp'})
changed: [webapp] => (item={'regex': 'username_here', 'value': 'staging_user'})
changed: [webapp] => (item={'regex': 'password_here', 'value': 'abcd1234'})

PLAY [database] **********************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [mysql]

PLAY RECAP ***************************************************************************************************************************************************
mysql                      : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
webapp                     : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```