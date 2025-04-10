pipeline {
    agent any
    options {
        timeout(time: 5, unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    environment {
        awsID = '339712874850'
        awsECRurl = 'dkr.ecr.us-east-1.amazonaws.com'
        project = 'expense'
        ENV = 'dev'
        component = 'frontend'
        awsRegion = 'us-east-1'
        awsCreds = 'aws-creds'
        appVersion = ''
        imageURL = ''
    }
    stages {
        stage('Read Version') {
            steps {
                script {
                    def packageJson = readJSON file: 'code/package.json'
                    appVersion = packageJson.version
                    echo "App Version: ${appVersion}"
                }
            }
        }
        stage('Setup') {
            steps {
                script {
                    imageURL = "${awsID}.${awsECRurl}/${project}/${ENV}/${component}"
                    echo "Image URL: ${imageURL}"
                }
            }
        }
        stage('Docker Build') {
            steps {
                sh """
                    docker build -t ${awsID}.${awsECRurl}/${project}/${ENV}/${component}:${appVersion} .
                    docker images
                """
            }
        }
        stage('Docker Push') {
            steps {
                withAWS(region: "${awsRegion}", credentials: "${awsCreds}") {
                    sh """
                        aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${awsID}.${awsECRurl}
                        docker push ${awsID}.${awsECRurl}/${project}/${ENV}/${component}:${appVersion}
                    """
                }
            }
        }
        stage('Deploy') {
            steps {
                withAWS(region: "${awsRegion}", credentials: "${awsCreds}") {
                    sh """
                        aws eks update-kubeconfig --region ${awsRegion} --name expense-dev
                        cd helm
                        sed -i 's/IMAGE_VERSION/${appVersion}/g' values-${ENV}.yaml
                        helm upgrade --install ${component} -f values-${ENV}.yaml .
                    """
                }
            }
        }
    }
    post {
        always {
            deleteDir()
        }
    }
}
