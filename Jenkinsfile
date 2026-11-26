def remote = [:]
pipeline {
    agent {
        label 'yandex'
    }
    parameters {
        string(name: 'VERSION', defaultValue: 'latest')
        booleanParam(name: 'RUN_TESTS', defaultValue: true)
        gitParameter(
          name: 'REVISION',
          type: 'PT_BRANCH',
          branchFilter: 'origin/(.*)',
          defaultValue: 'main',
          selectedValue: 'DEFAULT'
        )
    }
    environment {
        GIT_URL = "git@github.com:anestesia001/dos-37-cdcd_01.git"
        HOST = "158.160.223.144"
        PRJ_NAME = "diary"
        DOCKER_TOKEN = credentials('docker_token')
        DOCKER_REPO = "anestesia01/dos-37"
    }

    stages {
        stage('Configure credentials') {
            steps {
                withCredentials([
                    sshUserPrivateKey(credentialsId: 'jenkins-agent-key',
                    keyFileVariable: 'private_key',
                    usernameVariable: 'username'
                    )]
                ) {
                    script {
                        remote.name = "${env.HOST}"
                        remote.host = "${env.HOST}"
                        remote.user = "$username"
                        remote.identity = readFile "$private_key"
                        remote.allowAnyHosts = true
                    }
                }
            }
        }
        stage('Checkout repo') {
            steps {
                git branch: ${env.REVISION}, url: ${env.GIT_URL}, credentialsId: 'jenkins-agent-key'
            }
        }
        stage('Build docker image') {
            steps {
                script {
                    sh """
                        docker build -t "${env.DOCKER_REPO}:${env.PRJ_NAME}-${params.REVISION}" .
                        docker login -u anestesia01 -p "${env.DOCKER_TOKEN}"
                        docker push "${env.DOCKER_REPO}:${env.PRJ_NAME}-${params.REVISION}"
                    """
                }
            }
        }
        stage('Tests') {
            when {
                expression {
                    params.RUN_TESTS
                }
            }
            steps {
                sh """
                    docker run --rm "${env.DOCKER_REPO}:${env.PRJ_NAME}-${params.REVISION}" npm test
                """
            }
        }
        stage('Deploy') {
            steps {
                script {
                    sshCommand remote: remote, command: """
                        docker pull "${env.DOCKER_REPO}:${env.PRJ_NAME}-${params.REVISION}"
                        docker run -d -it -p 3000:3000 "${env.DOCKER_REPO}:${env.PRJ_NAME}-${params.REVISION}"
                    """
                }
            }
        }
        stage('Check') {
            steps {
                script {
                    sh """
                        curl -s -o /dev/null -w "%{http_code}" ${env.HOST}:3000
                    """
                }
            }
        }
    }
}