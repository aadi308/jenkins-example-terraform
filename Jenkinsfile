pipeline {
  agent any
  options {
    skipDefaultCheckout(true)
  }
  stages{
    stage('clean workspace') {
      steps {
        cleanWs()
      }
    }
    stage('checkout') {
      steps {
        checkout scm
      }
    }
    stage('terraform-init') {
      steps {
        withAWS(credentials: 'aadi_aws', region: 'us-east-2') {
          sh '''
           terraform init
         '''
         }
      }
    }
     stage('TF lint') {
           agent {
               docker {
                   image "ghcr.io/terraform-linters/tflint"
                   args '-i --entrypoint='
                 }
           }
           steps {
	     sh '''
               tflint . --no-color
             '''
	   }
    }
    stage('tfsec') {
      agent {
        docker {
          image 'tfsec/tfsec-ci'
          reuseNode true
        }
      }
      steps {
        sh '''
          tfsec . --no-color
        '''
      }
    }
    stage('terraform-apply-and-destroy') {
      steps {
        withAWS(credentials: 'aadi_aws', region: 'us-east-2') {
          sh '''
	   terraform apply -auto-approve -no-color
	   terraform destroy -auto-approve -no-color
         '''
         }
      }
    }
  }
  post {
    sucess {
            slackSend channel: 'jenkin_notifi', message: "status is:- ${currentBuild.currentResult} ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
           
 }
   }
  }

