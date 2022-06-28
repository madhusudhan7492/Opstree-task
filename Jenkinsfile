@Library('github.com/releaseworks/jenkinslib') _
pipeline {
  agent any
   stages {

      stage ('Checkout SCM'){
        steps {
          checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'Github_creds', url: 'https://github.com/madhusudhan7492/Opstree-task.git']]])
        }
      }

      withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'aws-key', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']]) {
        AWS("--region=us-east-1 ec2 describe-instances")
    }

    //   stage('Describe the instance'){
    //     steps{
    //         script{
    //             sh "aws ec2 describe-instances"

    //         }
            
    //     }
    //   }    
         
   } 
}