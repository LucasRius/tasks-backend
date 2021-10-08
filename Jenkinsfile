pipeline{
    agent any
    stages{
        stage('Build Backend'){
            steps{
                sh 'mvn clean package -DskipTests=true'
            }
        }
            stage('Unit Testes'){
                steps{
                    sh 'mvn test'
                }
        }
            stage('Sonar Analysis'){
                environment{
                    scannerHome = tool 'SONAR_SCANNER'
                }
                steps{
                    withSonarQubeEnv('SONAR_LOCAL'){
                        sh "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=83e0ca00d242691576ead3bd0964020b85d03cf1 -Dsonar.java.binaries=target"
                    }
                    
                }
        }
        stage('Quality Gate'){
            steps{
                sleep(7)
                timeout(time:1, unit:'MINUTES'){
                    waitForQualityGate abortPipeline: true
                }
                
            }    
        }
         stage('Deploy Backend'){
            steps{                
                deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'                
            }    
        }
        stage('API Test'){
            steps{
                dir('api-test'){
                    git 'https://github.com/LucasRius/tasks-api-test.git'
                    sh 'mvn test' 
                }                              
            }    
        }
    }
}

