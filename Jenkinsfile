pipeline{
    agent any
    tools{
        maven "MAVEN3"
        jdk "OracleJDK8"
    }
    
    environment {
        registryCredential = 'ecr:us-east-1:awscreds'
        appRegistry = "845527935813.dkr.ecr.us-east-1.amazonaws.com/vprofileappimg"
        vprofileRegistry = "https://845527935813.dkr.ecr.us-east-1.amazonaws.com"
        cluster = "vprofile"
        service = "vprofileappsvc"
    }

    stages{
        stage ("Code Fetch") {
            steps{
                git branch: "docker", url:"https://github.com/devopshydclub/vprofile-project.git"
            }
        }

        stage("Code Test"){
            steps{
            sh 'mvn test'
            }
        }

        stage ("Code Analysis"){
            steps{
                sh 'mvn checkstyle:checkstyle'
            }
            post{   
            success{
                echo "Generated Analysis Result"
            }
           
        }

        }

        stage('Sonar Analysis'){
            environment{
                scannerHome = tool 'sonar4.7'
            }
            steps{
                  withSonarQubeEnv('sonar') {
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


        
    stage('Build App Image') {
       steps {
       
         script {
                dockerImage = docker.build( appRegistry + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
             }

     }
    
    }

    stage('Upload App Image') {
          steps{
            script {
              docker.withRegistry( vprofileRegistry, registryCredential ) {
                dockerImage.push("$BUILD_NUMBER")
                dockerImage.push('latest')
              }
            }
          }
    

    }

    stage('Deploy to ecs') {
          steps {
        withAWS(credentials: 'awscreds', region: 'us-east-2') {
          sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
        }
      }
     }

    }

}
