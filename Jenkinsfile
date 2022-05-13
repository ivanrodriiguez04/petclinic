node {
  properties([pipelineTriggers([pollSCM('5 * * * *')]),[$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator', daysToKeepStr: '2', numToKeepStr: '5']]]);

  stage("Checkout") {
    deleteDir()
    checkout scm
  }

  stage("Build") {
      docker.image('maven').inside {
          sh "mvn package"
      }
  }
 
  stage("Report tests") {
      junit allowEmptyResults: true, testResults: '**/TEST-*.xml'
  }
  docker.image('chriscamicas/awscli-awsebcli').inside {
    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'AWS', usernameVariable: 'ivanrodriiguez04', passwordVariable: 'Covermanager2022S']]) {

      stage('Prepare environment') {
        sh 'eb init continuous-deployment-demo -p "64bit Amazon Linux 2017.09 v2.6.4 running Java 8" --region "eu-east-1" '

        try {
          sh 'eb create jenkins-env --single'
        } catch(e) {
          echo "Error while creating environment, continue..., cause: " + e
        }
        sh 'eb use jenkins-env'
        sh 'eb setenv SERVER_PORT=5000'
      }
      stage('Deploy') {
        sh 'eb deploy'
        sh 'eb status'
      }
    }
  }

}
