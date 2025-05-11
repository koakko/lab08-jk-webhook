pipeline {
    agent none
    triggers {
        githubPush()
    }
    environment {
        DH = credentials('dh-credentials')
    }
    stages {
        stage('Checkout') {
            agent { label 'master' }
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'git@github.com:koakko/lab08-jk-webhook.git',
                        credentialsId: 'gt-ssh'
                    ]]
                ])
            }
        }
        stage('build and push frontend image') {
            when {
                branch 'main'
            }
            agent {
                label 'master'
            }
            steps {
                dir('frontend') {
                    sh '''
                    if [ "$(docker image ls | grep fend)" ]; then
                    docker rmi -f koak/lab08-webhook:frontend || true
                    docker rmi -f fend
                    fi
                    docker logout
                    docker login -u $DH_USR -p $DH_PSW
                    docker build -t fend .
                    docker tag fend koak/lab08-webhook:frontend
                    docker push koak/lab08-webhook:frontend
                    '''
                }
            }
        }
        stage('build and push backend image') {
            when {
                branch 'main'
            }
            agent {
                label 'master'
            }
            steps {
                dir('backend') {
                    sh '''
                        if [ "$(docker image ls | grep bend)" ]; then
                        docker rmi -f koak/lab08-webhook:backend || true
                        docker rmi -f bend
                        fi
                        docker build -t bend .
                        docker tag bend koak/lab08-webhook:backend
                        docker push koak/lab08-webhook:backend
                    '''
                }
            }
        }
        stage('deploy frontend') {
            when {
                branch 'main'
            }
            agent {
                label 'frontend-agent'
            }
            steps {
                sh '''
                if [ "$(docker ps -a -q -f name=cfend)" ]; then
                docker rmi -f koak/lab08-webhook:frontend || true
                docker stop cfend
                docker rm -f cfend
                fi
                docker logout
                docker login -u $DH_USR -p $DH_PSW
                docker pull koak/lab08-webhook:frontend
                docker run -d -p 80:80 --name cfend koak/lab08-webhook:frontend
                '''
            }
        }
        stage('deploy backend') {
            when {
                branch 'main'
            }
            agent {
                label 'backend-agent'
            }
            steps {
                sh '''
                    if [ "$(docker ps -a -q -f name=cbend)" ]; then
                    docker rmi -f koak/lab08-webhook:backend || true
                    docker stop cbend
                    docker rm -f cbend
                    fi
                    docker logout
                    docker login -u $DH_USR -p $DH_PSW
                    docker pull koak/lab08-webhook:backend
                    docker run -d -p 5000:5000 --name cbend koak/lab08-webhook:backend
                '''
            }
        }
        stage('Checkout dev branch')
        when {
            branch 'dev'
        }
        agent {
            label 'frontend-agent'
        }
        steps {
            dir('frontend') {
                sh '''
                    if [ "$(docker ps -a -q -f name=cfend)" ]; then
                    docker rmi -f koak/lab08-webhook:frontend || true
                    docker rmi -f fend || true
                    docker stop cfend || true
                    docker rm -f cfend
                    fi
                    docker logout
                    docker login -u $DH_USR -p $DH_PSW
                    docker build -t fend .
                    docker tag fend koak/lab08-webhook:frontend
                    docker push koak/lab08-webhook:frontend
                    docker run -d -p 80:80 --name cfend koak/lab08-webhook:frontend
                '''
            }
        }
    }
    post {
        always {
            node('master') {
                sh 'echo "Your pipeline is finish!!"'
            }
        }
        success {
            node('master') {
                sh 'echo "Your pipeline is Successfull, Browse "http://localhost:80""'
            }
        }
        failure {
            node('master') {
                sh 'echo "Your pipeline is fail!"'
            }
        }
        }
    }