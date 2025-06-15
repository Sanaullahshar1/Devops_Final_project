pipeline {
    agent any

    environment {
        AZURE_SUBSCRIPTION_ID = credentials('azure-subscription-id')
        AZURE_CLIENT_ID = credentials('azure-client-id')
        AZURE_CLIENT_SECRET = credentials('azure-client-secret')
        AZURE_TENANT_ID = credentials('azure-tenant-id')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Terraform Init') {
            steps {
                dir('terraform') {
                    sh '''
                    terraform init \
                      -backend-config="subscription_id=$AZURE_SUBSCRIPTION_ID" \
                      -backend-config="client_id=$AZURE_CLIENT_ID" \
                      -backend-config="client_secret=$AZURE_CLIENT_SECRET" \
                      -backend-config="tenant_id=$AZURE_TENANT_ID"
                    '''
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                dir('terraform') {
                    sh '''
                    terraform apply -auto-approve \
                      -var="azure_subscription_id=$AZURE_SUBSCRIPTION_ID" \
                      -var="azure_client_id=$AZURE_CLIENT_ID" \
                      -var="azure_client_secret=$AZURE_CLIENT_SECRET" \
                      -var="azure_tenant_id=$AZURE_TENANT_ID"
                    '''
                }
                script {
                    def tfOutput = sh(script: 'cd terraform && terraform output -raw public_ip', returnStdout: true).trim()
                    env.TF_OUTPUT_PUBLIC_IP = tfOutput
                    
                    writeFile file: 'ansible/inventory.ini', text: """
                    [web]
                    ${tfOutput}

                    [web:vars]
                    ansible_user=adminuser
                    ansible_ssh_private_key_file=${workspace}/ansible/id_rsa
                    ansible_python_interpreter=/usr/bin/python3
                    """
                }
            }
        }

        stage('Ansible Provision') {
            steps {
                dir('ansible') {
                    sh 'ansible-playbook -i inventory.ini install_web.yml'
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    def response = sh(script: "curl -s http://${env.TF_OUTPUT_PUBLIC_IP}", returnStdout: true).trim()
                    if (!response.contains('Welcome to the DevOps Project')) {
                        error('Deployment verification failed')
                    } else {
                        echo "Deployment verified successfully at http://${env.TF_OUTPUT_PUBLIC_IP}"
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed'
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}
