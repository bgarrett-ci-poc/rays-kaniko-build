pipeline {
    agent none
    stages {
        stage('parallel'){
            parallel {
                stage('maven') {
                    agent {
                        kubernetes {
                            yaml '''
                                apiVersion: v1
                                kind: Pod
                                spec:
                                  containers:
                                  - name: maven
                                    image: maven:3.9.3-eclipse-temurin-11
                                    command:
                                    - sleep
                                    args:
                                    - infinity
                            '''
                        }
                    }
                    steps {
                        checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: '92843b3c-fdea-4d2d-a36b-2b1263933aa0', url: 'https://github.com/rkivisto/simple-maven-project-with-tests.git']])
                        container('maven') {
                            sh 'mvn -B -ntp -Dmaven.test.failure.ignore verify'
                        }
                        junit '**/target/surefire-reports/TEST-*.xml'
                    }
                }
                stage('build-image'){
                    agent {
                        kubernetes {
                            yaml '''
                                kind: Pod
                                spec:
                                  containers:
                                  - name: kaniko
                                    image: gcr.io/kaniko-project/executor:debug
                                    command:
                                    - sleep
                                    args:
                                    - infinity
                            '''
                        }
                    }
                    stages {
                        stage('Dockerfile'){
                            steps {
                                sh '''cat >Dockerfile <<EOF
FROM cloudbees/cloudbees-core-agent:${AGENT_VERSION}
#USER root
#RUN dnf update -y && dnf install -y maven-3.5.4 && dnf clean all && rm -rf /var/cache/dnf
#USER jenkins
#EOF
'''
                            }
                        }
                        stage('Build with Kaniko') {
                            steps {
                                container('kaniko') {
                                        sh '''#!/busybox/sh
                                              /kaniko/executor \
                                              --context `pwd` \
                                              --cache=false \
                                              --no-push
                                        '''
                                        sh 'ls'
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}
