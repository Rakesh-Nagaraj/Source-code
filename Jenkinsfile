pipeline{
    agent any
    tools{
        maven "MAVEN3"
        jdk "OracleJDK8"
    }
    stages{
        stage ("Code Fetch") {
            steps{
                git branch: "vp-rem", url:"https://github.com/devopshydclub/vprofile-project.git"
            }
        }

        stage("Code Build"){
            steps{
            sh 'mvn install -DskipTests'
            }
        post{   
            success{
                echo "Now Archiving it"
                archiveArtifacts artifacts: '**/target/*.war'
            }
           
        }

        }
       
        stage ("Unit Test"){
            steps{
                sh 'mvn test'
            }

        }

        stage ("Code Analysis"){
            steps{
                sh 'mvn checkstyle:checkstyle'
            }

        }

        stage('Sonar Analysis'){
            environment{
                scannerHome = tool 'sonar4.7'
            }
            steps{
                  withSonarQubeEnv('sonar-pro') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''

            }
        }

        }

        stage('Quality gates'){
            steps{
                timeout(time: 10, unit: 'MINUTES') {
               waitForQualityGate abortPipeline: true
                }

        }
    
    }
}