pipeline {
    agent any

    environment {
        HARBOR_PATH = "13.201.3.250/crm/${params.IMAGE_NAME}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: params.BRANCH_NAME, credentialsId: 'github-cred', url: "${params.GITHUB_URL}"
            }
        }

        stage('Build docker image') {
            steps {
                dir(params.DIR_NAME){
                    sh "docker build -t ${params.IMAGE_NAME} ."
                }
            }
        }
            
        stage('docker tag image') {
            steps {
                sh """
                docker tag ${params.IMAGE_NAME} ${env.HARBOR_PATH}:$BUILD_NUMBER
                docker tag ${params.IMAGE_NAME} ${env.HARBOR_PATH}:latest
                """
            }
        }

        stage('push image to harbor') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'harbor-cred', passwordVariable: 'HARBOR_PASSWORD', usernameVariable: 'HARBOR_USERNAME')]) {
                    sh """
                    docker push ${env.HARBOR_PATH}:$BUILD_NUMBER
                    docker push ${env.HARBOR_PATH}:latest
                    """
                }
            }
        }

        stage('clean up local docker images') {
            steps {
                sh """
                docker rmi ${params.IMAGE_NAME}
                docker rmi ${env.HARBOR_PATH}:$BUILD_NUMBER
                docker rmi ${env.HARBOR_PATH}:latest
                """
            }
        }
        
        stage('deploy in kubernetes'){
            steps{
                sshagent(['test-ssh']) {
                sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@65.0.100.173 '
                    cd crm-backend
                    helm install ${params.IMAGE_NAME} . -f values/values.${params.IMAGE_NAME}.yml
                    '
                """
                }
            }
        }
    }
}
