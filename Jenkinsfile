pipeline{
    agent any
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
                        timeout(5) {
                            def qg = waitForQualityGate()
                            if (qg.status='OK') {
                                error "the quality gate treshold: ${qg.status}"
                            }
                        }    
                    }
                }
            }
        }
    }
}
