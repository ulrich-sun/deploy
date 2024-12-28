pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('aws_access_key_id')
        AWS_SECRET_ACCESS_KEY = credentials('aws_secret_access_key')
    }
    stages {
        stage('Build docker EC2') {
            agent {
                docker {
                    image 'jenkins/jnlp-agent-terraform'
                }
            }
            steps {
                script {
                    dir('/var/jenkins_home/workspace/deploy') {
                        sh '''
                            mkdir -p ~/.aws
                            echo "[default]" > ~/.aws/credentials
                            echo "aws_access_key_id=$AWS_ACCESS_KEY_ID" >> ~/.aws/credentials
                            echo "aws_secret_access_key=$AWS_SECRET_ACCESS_KEY" >> ~/.aws/credentials
                            chmod 400 ~/.aws/credentials
                            cd 02_terraform/
                            terraform init
                            terraform apply -var="stack=docker" -auto-approve
                        '''
                    }
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
                    dir('/var/jenkins_home/workspace/deploy') {
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
}
