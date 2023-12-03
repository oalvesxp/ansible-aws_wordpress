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
                script {
                    cd ~/ansible/c-wordpress
                    ansible -m ping all --vault-password-file ./files/.vault.txt
                }
            }
        }
    }
}