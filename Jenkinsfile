#!/usr/bin/env groovy

properties([
    parameters([
        string(name: 'IMAGE_TAG',
               description: 'Docker image tag to deploy (e.g., latest, v1.0.0)',
               defaultValue: 'latest')
    ])
])

pipeline {
    agent any

    environment {
        APP_PORT = '80'
        APP_VERSION = params.IMAGE_TAG
        VAULT_PASSWORD = credentials('ansible-vault-pass')
    }

    stages {
        stage('User Input') {
            steps {
                script {
                    echo "Selected Docker image tag: ${params.IMAGE_TAG}"
                }
            }
        }

        stage('Deploy VM') {
            steps {
                script {
                    sh """
                        cd ansible
                        if [[ ! -f cw2_app_key.pem ]]; then
                            ansible-playbook ./provision_key_security_group.yml
                        fi
                        ansible-playbook ./provision_staging.yml
                    """
                }
            }
        }

        stage('Verify Application') {
            steps {
                script {
                    // Replace with your actual application URL
                    def app_url = "http://${env.PUBLIC_IP}:${APP_PORT}"

                    echo "Verifying application at ${app_url}"

                    // Wait for application to be ready
                    sleep(time: 6, unit: 'SECONDS')

                    // Perform health check
                    sh """
                        if curl -sSf $app_url > /dev/null; then
                            echo "Application is responding successfully"
                        else
                            echo "Application check failed"
                            exit 1
                        fi
                    """
                }
            }
        }
    }
}