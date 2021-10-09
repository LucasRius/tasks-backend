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
        stage('Frontend Deploy'){
            steps{                
                dir('frontend-test'){
                    git 'https://github.com/LucasRius/tasks-frontend.git'
                    sh 'mvn clean package'
                    deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'                     
                }                              
            }    
        }
         stage('Functional Test'){
            steps{
                dir('functional-test'){
                    git 'https://github.com/LucasRius/functional-tasks.git'
                    sh 'mvn test' 
                }                              
            }    
        }
        stage('Deploy Prod'){
            steps{
                sh 'docker-compose build'
                sh 'docker-compose up -d'
            }
        }
        stage('Health Check'){
            steps{
                sleep(10)
                dir('functional-test'){                    
                    sh 'mvn verify -Dskip.surefire.tests' 
                }                              
            }    
        }
    }
    post{
        always{
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml, api-test/target/surefire-reports/*.xml, functional-test/target/surefire-reports/*.xml, functional-test/target/failsafe-reports/*.xml'
        }
        unsuccessful {
            emailext attachLog: true, body: 'Veja o log', subject: 'Build $BUILD_NUMBER falhou', to: 'lucasrius.aws+jenkins@gmail.com'
        }
    }
}

