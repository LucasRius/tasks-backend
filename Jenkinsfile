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
                timeout(time:1, unit:'MINUTES'){
                    waitForQualityGate abortPipeline: true
                }
                
            }    
        }
    }
}

