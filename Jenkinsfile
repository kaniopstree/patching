pipeline {

    agent any

    parameters {

        string(
            name: 'TARGET_HOST',
            defaultValue: 'localhost',
            description: 'Target Server IP/Hostname'
        )

        string(
            name: 'TARGET_PORT',
            defaultValue: '2222',
            description: 'SSH Port'
        )

        string(
            name: 'SSH_USER',
            defaultValue: 'kanimozhi',
            description: 'SSH Username'
        )

        string(
            name: 'PACKAGE_NAME',
            defaultValue: 'nginx',
            description: 'Package Name'
        )

        string(
            name: 'PACKAGE_VERSION',
            defaultValue: '',
            description: 'Leave blank to install latest available version'
        )
    }

    stages {

        stage('Connectivity Check') {

            steps {

                echo "========== CONNECTIVITY CHECK =========="

                sshagent(['patching-server']) {

                    sh """
                    ssh \
                    -o StrictHostKeyChecking=no \
                    -p ${params.TARGET_PORT} \
                    ${params.SSH_USER}@${params.TARGET_HOST} \
                    "
                    hostname
                    whoami
                    "
                    """
                }
            }
        }

        stage('Current Installed Version') {

            steps {

                echo "========== CURRENT INSTALLED VERSION =========="

                sshagent(['patching-server']) {

                    sh """
                    ssh \
                    -o StrictHostKeyChecking=no \
                    -p ${params.TARGET_PORT} \
                    ${params.SSH_USER}@${params.TARGET_HOST} \
                    "
                    dpkg -l | grep ${params.PACKAGE_NAME} || true
                    "
                    """
                }
            }
        }

        stage('Repository Versions') {

            steps {

                echo "========== AVAILABLE REPOSITORY VERSIONS =========="

                sshagent(['patching-server']) {

                    sh """
                    ssh \
                    -o StrictHostKeyChecking=no \
                    -p ${params.TARGET_PORT} \
                    ${params.SSH_USER}@${params.TARGET_HOST} \
                    "
                    apt-cache madison ${params.PACKAGE_NAME}
                    "
                    """
                }
            }
        }

        stage('Upgrade/Downgrade Package') {

            steps {

                echo "========== UPGRADE / DOWNGRADE PACKAGE =========="

                sshagent(['patching-server']) {

                    script {

                        if (params.PACKAGE_VERSION.trim() == "") {

                            echo "Installing Latest Available Version"

                            sh """
                            ssh \
                            -o StrictHostKeyChecking=no \
                            -p ${params.TARGET_PORT} \
                            ${params.SSH_USER}@${params.TARGET_HOST} \
                            "
                            sudo apt update
                            sudo apt install -y ${params.PACKAGE_NAME}
                            "
                            """

                        } else {

                            echo "Requested Version : ${params.PACKAGE_VERSION}"

                            echo "Validating Requested Version"

                            sh """
                            ssh \
                            -o StrictHostKeyChecking=no \
                            -p ${params.TARGET_PORT} \
                            ${params.SSH_USER}@${params.TARGET_HOST} \
                            "
                            apt-cache madison ${params.PACKAGE_NAME} | grep '${params.PACKAGE_VERSION}'
                            "
                            """

                            echo "Installing Target Version"

                            sh """
                            ssh \
                            -o StrictHostKeyChecking=no \
                            -p ${params.TARGET_PORT} \
                            ${params.SSH_USER}@${params.TARGET_HOST} \
                            "
                            sudo apt update

                            sudo apt install \
                                --allow-downgrades \
                                --allow-change-held-packages \
                                -y \
                                ${params.PACKAGE_NAME}=${params.PACKAGE_VERSION}
                            "
                            """
                        }
                    }
                }
            }
        }
                stage('Verification') {

            steps {

                echo "========== VERIFY INSTALLED PACKAGE =========="

                sshagent(['patching-server']) {

                    sh """
                    ssh \
                    -o StrictHostKeyChecking=no \
                    -p ${params.TARGET_PORT} \
                    ${params.SSH_USER}@${params.TARGET_HOST} \
                    "
                    echo '=========================================='
                    echo 'Installed Package Details'
                    echo '=========================================='

                    dpkg -l | grep ${params.PACKAGE_NAME}

                    echo

                    echo '=========================================='
                    echo 'Repository Package Information'
                    echo '=========================================='

                    apt-cache policy ${params.PACKAGE_NAME}

                    echo

                    echo '=========================================='
                    echo 'Application Version'
                    echo '=========================================='

                    ${params.PACKAGE_NAME} -v || true
                    "
                    """
                }
            }
        }

        stage('Patch Summary') {

            steps {

                script {

                    def version = params.PACKAGE_VERSION?.trim() ? params.PACKAGE_VERSION : "LATEST"

                    echo """
====================================================

                PATCH SUMMARY

====================================================

Target Host        : ${params.TARGET_HOST}

SSH User           : ${params.SSH_USER}

SSH Port           : ${params.TARGET_PORT}

Package            : ${params.PACKAGE_NAME}

Requested Version  : ${version}

Operation          : Upgrade / Downgrade

Status             : SUCCESS

====================================================
"""
                }
            }
        }
    }

    post {

        success {

            echo "=========================================="
            echo "Package patching completed successfully."
            echo "=========================================="
        }

        failure {

            echo "=========================================="
            echo "Package patching failed."
            echo "Please check the Jenkins Console Output."
            echo "=========================================="
        }

        always {

            cleanWs()
        }
    }
}
