pipeline {
    agent any
    options {
        timeout(time: 5, unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    environment {
        project = 'expense'
        component = 'frontend'
        ENV = 'dev'
        awsID = '339712874850'
        awsRegion = 'us-east-1'
        awsCreds = 'aws-creds'
        awsECRurl = 'dkr.ecr.us-east-1.amazonaws.com'
        appVersion = ''
        imageURL = ''
    }
    parameters{
        booleanParam(name: 'deploy', defaultValue: false, description: 'Select for deploy')
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
                    docker build -t ${imageURL}:${appVersion} .
                    docker images
                """
            }
        }
        stage('Docker Push') {
            steps {
                withAWS(region: "${awsRegion}", credentials: "${awsCreds}") {
                    sh """
                        aws ecr get-login-password --region ${awsRegion} | docker login --username AWS --password-stdin ${awsID}.${awsECRurl}
                        docker push ${imageURL}:${appVersion}
                    """
                }
            }
        }
        stage('Deploy') {
            when {
                expression { params.deploy }
            }
            steps{
                build job: 'frontend-cd', parameters: [
                    string(name: 'version', value: "$appVersion"),
                    string(name: 'ENV', value: 'dev'),
                ], wait: true
            }
        }
    }
    post {
        always {
            deleteDir()
        }
    }
}
