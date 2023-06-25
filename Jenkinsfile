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

    stage("UploadArtifact"){
            steps{
                nexusArtifactUploader(
                  nexusVersion: 'nexus3',
                  protocol: 'http',
                  nexusUrl: '172.31.18.28:8081',
                  groupId: 'QA',
                  version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                  repository: 'vprofile-repo',
                  credentialsId: 'nexuslogin',
                  artifacts: [
                    [artifactId: 'vproapp',
                     classifier: '',
                     file: 'target/vprofile-v2.war',
                     type: 'war']
                  ]
                )
            }
        }
    }
    post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#jenkinscicd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}
