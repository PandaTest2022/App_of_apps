def backendDockerTag=""
def frontendDockerTag=""
def frontendImage="panda20222/frontend"
def backendImage="panda20222/backend"
def dockerRegistry=""
def registryCredentials="dockerhub"

pipeline {
    agent {
        label'agent'
    }
    tools{
        terraform 'Terraform'
    }
    parameters {
        string(name: 'backendDockerTag', defaultValue: '', description: 'Backend docker image tag')
        string(name: 'frontendDockerTag', defaultValue: '', description: 'Frontend docker image tag')
    }
    stages{
        stage('GetCode') {
            steps{
               checkout scm
            }
        }
        stage('Clean runningcontainers') {
            steps{
                sh "docker rm-f frontend backend"
            }
        }
        stage('Adjust version') {
            steps {
                script{
                    backendDockerTag = params.backendDockerTag.isEmpty() ? "latest" : params.backendDockerTag
                    frontendDockerTag = params.frontendDockerTag.isEmpty() ? "latest" : params.frontendDockerTag
                    
                    currentBuild.description = "Backend: ${backendDockerTag}, Frontend: ${frontendDockerTag}"
                }
            }
        }
        stage('Deploy application') {
            steps {
                script {
                    withEnv(["FRONTEND_IMAGE=$frontendImage:$frontendDockerTag", 
                             "BACKEND_IMAGE=$backendImage:$backendDockerTag"]) {
                       docker.withRegistry("$dockerRegistry", "$registryCredentials") {
                            sh "docker-compose up -d"
                        }
                    }
        stage('Selenium tests') {
            steps{
                sh"pip3 install-r test/selenium/requirements.txt"
                sh"python3 -m pytesttest/selenium/frontendTest.py"
                    }
                }
        stage('Run terraform') {
            steps {
                dir('Terraform') {                
                    git branch: 'main', url: 'https://github.com/PandaTest2022/Terraform.git'
                    withAWS(credentials:'AWS', region: 'us-east-1') {
                            sh 'terraform init && terraform apply -auto-approve -var-file="terraform.tfvars"'
                    } 
                }

            stage('Run Ansible') {
               steps {
                   script {
                        sh "ansible-galaxy install -r requirements.yml"
                        withEnv(["FRONTEND_IMAGE=$frontendImage:$frontendDockerTag", 
                                 "BACKEND_IMAGE=$backendImage:$backendDockerTag"]) {
                            ansiblePlaybook inventory: 'inventory', playbook: 'playbook.yml'
                     }
                }

            }
        }
    } 
}
