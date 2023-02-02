pipeline{
    agent any
    environment{
        VERSION = "${BUILD_ID}"
    }
    stages{
        stage("Sonar code check"){
            agent {
                docker {
                    image 'openjdk:11'
                }
            }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-pwd') {
                        sh "chmod +x gradlew"
                        sh "./gradlew sonarqube"
                    }
                        timeout(5) {
                            def qg = waitForQualityGate()
                            if (qg.status != 'OK') {
                                error "the quality gate treshold: ${qg.status}"
                            }
                        }
                }
            }
        }
        stage("docker build & push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                        sh '''
                            docker build -t 35.200.144.65:8083/springapp:${VERSION} .
                            docker login -u admin -p docker_password 35.200.144.65:8083
                            docker push 35.200.144.65:8083/springapp:${VERSION}
                            docker rmi 35.200.144.65:8083/springapp:${VERSION}
                        '''
                    }

                }
            }
        }
    }
}
