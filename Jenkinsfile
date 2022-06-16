pipeline{
    agent any
    stages{
        stage ('Build Backend') {
            steps{
                bat 'mvn clean package -DskipTests=true'
            }
        }
        stage ('Unit Tests') {
            steps{
                bat 'mvn test'
            }
        }
        stage ('Sonar Analyses') {
            environment{
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps{
                withSonarQubeEnv ('SONAR_LOCAL') {
                    bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=845087f5e8a81e4edd453b8a6eebd828edf27251 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**/TaskBackendApplication.java,**/RootController.java"
                }
            }
        }
        stage ('Quality Gate') {
            steps{             
                sleep(10)
                timeout(time:1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline:true                   
                }
            }
        }
    }
}

