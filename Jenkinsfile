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
    }
}