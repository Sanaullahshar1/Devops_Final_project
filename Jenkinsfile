pipeline {
    agent any

    environment {
        SSH_PRIVATE_KEY_ID = 'your-ssh-key-credential-id' // Replace this with your Jenkins credential ID
    }

    stages {
        stage('Terraform Init & Apply') {
            steps {
                dir('terraform') {
                    sh 'terraform init'
                    sh 'terraform apply -auto-approve'
                }
            }
        }

        stage('Get VM IP') {
            steps {
                script {
                    dir('terraform') {
                        env.VM_PUBLIC_IP = sh(script: "terraform output -raw public_ip", returnStdout: true).trim()
                        echo "VM Public IP is ${env.VM_PUBLIC_IP}"
                    }
                }
            }
        }

        stage('Deploy via Ansible') {
            steps {
                sshagent(credentials: [env.SSH_PRIVATE_KEY_ID]) {
                    sh """
                    ansible-playbook -i '${env.VM_PUBLIC_IP},' ansible/install_web.yml -u adminuser --private-key ~/.ssh/id_rsa
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    sh "curl http://${env.VM_PUBLIC_IP}"
                }
            }
        }
    }
}