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
                        sh "./gradlew sonarqube --debug"
                        sh "./gradlew sonarqube"
                        timeout(5) {
                        def qg = withForQualityGate()
                            if (qg.status != 'OK'){
                             error "quality gate status : ${qg.status}"           
                            }
                        }
                        
                    }    
                }
            }
        }
    }
}
