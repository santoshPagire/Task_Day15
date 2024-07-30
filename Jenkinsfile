pipeline {
    agent any
    environment {
        ANSIBLE_SUDO_PASSWORD = credentials('Ansible')
        registry = 'docker.io'  
        registryCredential = 'docker' 
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/santoshPagire/Task_Day15.git', branch: 'main'
            }
        }

        stage('build image') {
            steps{
                script{
                    docker.withRegistry('', registryCredential){
                        def customImage = docker.build("santoshpagire/mynginx-app:latest")
                        customImage.push()
                       

                    }

                }
            }
        }
        stage('Run Playbook') {
            steps{
               sh """
                   export ANSIBLE_SUDO_PASSWORD=${ANSIBLE_SUDO_PASSWORD}
                   ansible-playbook playbook.yml -i inventory -u jenkins --extra-vars "ansible_become_pass=${ANSIBLE_SUDO_PASSWORD}"
                """
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Pipeline succeeded.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }

}
 