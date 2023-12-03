pipeline {
    agent any

    stages {
        stage ('Start') {
            steps {
                echo "Starting pipeline..."
            }
        }

        stage ('Conection check') {
            steps {
                ansiblePlaybook become: true, credentialsId: 'ansible.pem', disableHostKeyChecking: true, installation: 'Ansible', inventory: '/etc/ansible/c-wordpress/inventory_hml_wordpress', playbook: '/etc/ansible/c-wordpress/playbook-all-ping.yml', vaultCredentialsId: 'ansible-vault', vaultTmpPath: ''
            }
        }

        stage ('Setup common') {
            steps {
                ansiblePlaybook become: true, credentialsId: 'ansible.pem', disableHostKeyChecking: true, installation: 'Ansible', inventory: '/etc/ansible/c-wordpress/inventory_hml_wordpress', playbook: '/etc/ansible/c-wordpress/playbook-all-common.yml', vaultCredentialsId: 'ansible-vault', vaultTmpPath: ''
            }
        }

        stage ('Setup all env') {
            steps {
                ansiblePlaybook become: true, credentialsId: 'ansible.pem', disableHostKeyChecking: true, installation: 'Ansible', inventory: '/etc/ansible/c-wordpress/inventory_hml_wordpress', playbook: '/etc/ansible/c-wordpress/playbook-all-apache2.yml', vaultCredentialsId: 'ansible-vault', vaultTmpPath: ''
                ansiblePlaybook become: true, credentialsId: 'ansible.pem', disableHostKeyChecking: true, installation: 'Ansible', inventory: '/etc/ansible/c-wordpress/inventory_hml_wordpress', playbook: '/etc/ansible/c-wordpress/playbook-all-php8.2-wp.yml', vaultCredentialsId: 'ansible-vault', vaultTmpPath: ''
                ansiblePlaybook become: true, credentialsId: 'ansible.pem', disableHostKeyChecking: true, installation: 'Ansible', inventory: '/etc/ansible/c-wordpress/inventory_hml_wordpress', playbook: '/etc/ansible/c-wordpress/playbook-all-mariadb-setup.yml', vaultCredentialsId: 'ansible-vault', vaultTmpPath: ''
            }
        }
    }
}