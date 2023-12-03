# ansible-aws_wordpress
Provisionando um ambiente simples para wordpress na AWS.

### Descrição do Projeto
---

O objetivo é configurar um ambiente para wordpress com ansible de maneira segura e com código reutilizável.

###  Projeto
---

* Configuração do ansible (ansible.cfg)
* Criação das roles.
  * **apache2-setup:** Instalação e configuração do apache2 no servidor web.
  * **mariadb-10.5-setup:** Instalação e configuração do mariadb-10.5 no servidor de banco.
  * **php8.2-setup:** Instalação e configuração do php8.2 no servidor de aplicação.
  * **wordpress-setup:** Instalação e configuração do wordpress no servidor de aplicação / web.
* Criação dos playbooks de execução.

### Softwares & versões
---

1. SO da Instância:
```
$ hostnamectl | grep -i "operating system"
Operating System: Debian GNU/Linux 12 (bookworm)
```

2. Versão do Ansible:
```
$ ansible --version
ansible [core 2.14.3]
  config file = None
  configured module search path = ['/home/oalves/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /home/oalves/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.11.2 (main, Mar 13 2023, 12:18:29) [GCC 12.2.0] (/usr/bin/python3)
  jinja version = 3.1.2
  libyaml = True
```