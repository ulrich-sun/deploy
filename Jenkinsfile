pipeline {
    agent none
    environment {
        AWS_ACCESS_KEY_ID = credentials('aws_access_key_id')
        AWS_SECRET_ACCESS_KEY = credentials('aws_secret_access_key')
    }
    stages {
        stage('Build k3s EC2') {
            agent {
                docker {
                    image 'jenkins/jnlp-agent-terraform'
                }
            }
            steps {
                script {
                    sh '''
                        mkdir -p ~/.aws
                        echo "[default]" > ~/.aws/credentials
                        echo -e "aws_access_key_id=$AWS_ACCESS_KEY_ID" >> ~/.aws/credentials
                        echo -e "aws_secret_access_key=$AWS_SECRET_ACCESS_KEY" >> ~/.aws/credentials
                        chmod 400 ~/.aws/credentials
                        cd 02_terraform/
                        terraform init
                        terraform apply -var="stack=kubernetes" -auto-approve
                    '''
                }
            }
        }
        stage('Ansible ping on kubernetes vm') {
            agent {
                docker {
                    image 'registry.gitlab.com/robconnolly/docker-ansible:latest'
                }
            }
            steps {
                script {
                    sh '''
                        cd 04_ansible/
                        chmod 600 docker.pem
                        ansible all -i inventory -m ping
                    '''
                }
            }
        }
    }
}
