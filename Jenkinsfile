pipeline{
    agent any
    stages{
        stage('git checkout'){
            steps{
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/drishti0907/phishing-detection-system.git']])
            }
        }
        stage('build docker image'){
            steps{
                sh 'docker build -t phisher-detector.azurecr.io/phishing-detection-system:latest .'
            }
        }
        stage('push image'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'ACR', passwordVariable: 'password', usernameVariable: 'username')]) {
                sh 'docker login -u ${username} -p ${password} phisher-detector.azurecr.io'
                sh 'docker push phisher-detector.azurecr.io/phishing-detection-system:latest'
                }
            }
        }
        stage('install Azure CLI'){
            steps{
                sh '''
                apk add py3-pip
                apk add gcc musl-dev python3-dev libffi-dev openssl-dev cargo make
                pip install --upgrade pip
                pip install azure-cli
                '''
            }
        }
        stage('deploy web app'){
            steps{
                withCredentials([azureServicePrincipal('AZURE_SERVICE_PRINCIPAL')]) {
                sh 'az login --service-principal -u ${AZURE_CLIENT_ID} -p ${AZURE_CLIENT_SECRET} --tenant ${AZURE_TENANT_ID}'
                }
                withCredentials([usernamePassword(credentialsId: 'ACR', passwordVariable: 'password', usernameVariable: 'username')]) {
                sh 'az webapp config container set --name phisher-dectection-system --resource-group project --docker-custom-image-name phisher-detector.azurecr.io/phishing-detection-system:latest --docker-registry-server-url https://phisher-detector.azurecr.io --docker-registry-server-user ${username} --docker-registry-server-password ${password}'
                }
            }
        }
    }
}