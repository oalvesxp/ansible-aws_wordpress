pipeline {
    agent any

    stages {
        stage ('Inicio') {
            steps {
                echo "Iniciando a Pipeline - Wordpress Setup"
            }
        }
        stage ('Teste de conexão') {
            steps {
                echo "Testando a conexão dos servidores..."
                ansiblePlaybook credentialsId: 'ansible.pem', disableHostKeyChecking: true, installation: 'Ansible', inventory: '/home/ubuntu/ansible/c-wordpress/inventory_hml_wordpress', playbook: '/home/ubuntu/ansible/c-wordpress/playbook-all-common.yml', vaultCredentialsId: 'ansible-vault', vaultTmpPath: ''
            }
        }
    }
}