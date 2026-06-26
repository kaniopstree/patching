pipeline {

    agent any

    stages {

        stage('Run Package Patching') {

            steps {

                sh """
                ansible-playbook \
                -i inventory/hosts.ini \
                playbooks/update-package.yml \
                -e HOSTS=${params.HOSTS} \
                -e PACKAGE=${params.PACKAGE} \
                -e VERSION='${params.VERSION}'
                """
            }
        }
    }

    post {

        success {
            echo "Package patching completed successfully."
        }

        failure {
            echo "Package patching failed."
        }
    }
}
