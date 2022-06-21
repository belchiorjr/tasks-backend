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
                    bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=ada03e9701281eba7b843d14b237eaec6bfd8c36 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**/TaskBackendApplication.java,**/RootController.java"
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
        stage ('Deploy Backend') {
            steps{             
                deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'               
            }
        }
        stage ('API Test') {
            steps{       
                dir('api-test') {
                    git branch: 'main', credentialsId: 'github_login', url: 'https://github.com/belchiorjr/tasks-api-test'
                    bat 'mvn test'
                }      
            }
        }
        stage ('Deploy Frontend') {
            steps{       
                dir('frontend') {
                    git branch: 'master', credentialsId: 'github_login', url: 'https://github.com/belchiorjr/tasks-frontend'
                    bat 'mvn clean package'
                    deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
                }      
            }
        }
        stage ('Functional Test') {
            steps{       
                dir('functional-test') {
                    git branch: 'main', credentialsId: 'github_login', url: 'https://github.com/belchiorjr/tasks-funcional-test'
                    bat 'mvn test'
                }      
            }
        }
        stage ('Deploy Prod') {
            steps{       
                bat 'docker-compose build'    
                bat 'docker-compose up -d'    
            }
        }
        stage ('Health Check') {
            steps{     
                sleep(5)  
                dir('functional-test') {
                    bat 'mvn verify -Dskip.surefire.tests'
                }      
            }
        }
    }
    post{
        always {
            junit keepLongStdio: true, testResults: 'target/surefire-reports/*.xml,api-test/target/surefire-reports/*.xml,functional-test/target/surefire-reports/*.xml,functional-test/target/failsafe-reports/*.xml' 
            archiveArtifacts artifacts: 'target/tasks-backend.war, frontend/target/tasks.war', followSymlinks: false, onlyIfSuccessful: true
        }
        unsuccessful {
            bat 'echo "envio de email com o log de erros"'
        }
        fixed {
            bat 'echo "envio de email com o log de sucesso"'
        }
    }
}