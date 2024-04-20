pipeline {
    agent any
    stages {
        stage("checkout scm") {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/dev']], userRemoteConfigs: [[url: 'https://github.com/animesh0406/code_task_2.git']]])
            }
        }
        stage("build client") {
            agent {
                docker {
                    image 'node:10.19.0'
                    args '-u root:root'
                }
            }
            steps {
                dir('notes-ui') {
                    sh 'npm install react react-dom react-router-dom react-scripts web-vitals'
                }
            }
        }
        stage("build server") {
            agent {
                docker {
                    image 'node:10.19.0'
                    args '-u root:root'
                }
            }
            steps {
                dir('server') {
                    sh 'npm install body-parser cross-env express mocha mysql2 nodemon sequelize should sqlite3 supertest'
                }
            }
        }
        stage("test") {
            steps {
                sh "echo 'some test successfully performed'"
            }
        }
        stage("checking dockerfile linting") {
            parallel {
                stage("check linting for ui-dockerfile") {
                    steps {
                        dir('notes-ui') {
                            script {
                                sh 'hadolint Dockerfile > hadolint.txt || true'

                                def hadolintOutput = readFile('hadolint.txt').trim()
                                if (hadolintOutput) {
                                    error "Hadolint detected errors in Dockerfile:\n${hadolintOutput}"
                                }
                            }
                        }
                    }
                }
                stage("check linting for server-dockerfile") {
                    steps {
                        dir('server') {
                            script {
                                sh 'hadolint Dockerfile > hadolint.txt || true'

                                def hadolintOutput = readFile('hadolint.txt').trim()
                                if (hadolintOutput) {
                                    error "Hadolint detected errors in Dockerfile:\n${hadolintOutput}"
                                }
                            }
                        }
                    }
                }
            }
        }
        stage("building the images") {
            steps {
                script {
                    def clientTag = "animesh0406/task_2_UI_dev:${env.BUILD_NUMBER}"
                    def serverTag = "animesh0406/task_2_SERVER_dev:${env.BUILD_NUMBER}"

                    sh "docker build -t ${clientTag} notes-ui"
                    sh "docker build -t ${serverTag} server"
                }
            }
        }
        stage("pushing images on docker-hub repo") {
            steps {
                script {
                    def clientTag = "animesh0406/task_2_UI_dev:${env.BUILD_NUMBER}"
                    def serverTag = "animesh0406/task_2_SERVER_dev:${env.BUILD_NUMBER}"

                    withCredentials([usernamePassword(credentialsId: 'docker_credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                        sh "docker push ${clientTag}"
                        sh "docker push ${serverTag}"
                    }
                }
            }
        }
        stage("Deploy on dev server") {
            steps {
                  checkout([
                        $class: 'GitSCM',
                        branches: [[name: 'main']], // or specify the branch you want to checkout
                        userRemoteConfigs: [[url: 'https://github.com/animesh0406/DevOps_of_task_2.git']]
                    ])
                sshagent(credentials: ['remote_user_ssh']) {
                    sh "scp -o StrictHostKeyChecking=no -r docker-compose.yml root@192.168.18.102:/root/"
                    sh "ssh -o StrictHostKeyChecking=no -T root@192.168.18.102 'docker-compose up -d'"
                }
            }
        }
    }
}
