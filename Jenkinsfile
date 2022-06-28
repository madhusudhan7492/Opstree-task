@Library('github.com/releaseworks/jenkinslib') _
pipeline {
  agent any
   stages {

      stage ('Checkout SCM'){
        steps {
          checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'Github_creds', url: 'https://github.com/madhusudhan7492/Opstree-task.git']]])
        }
      }

      stage("Describe the instance"){
        steps{
            withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'aws-key', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']]) {
        AWS("--region=us-east-1 ec2 describe-instances --query 'Reservations[*].Instances[*].{PublicIP:PublicIpAddress,Name:Tags[?Key=='Name']|[0].Value,Status:State.Name,InstanceID:InstanceId}' --filters Name=instance-state-name,Values=running --output table")
         }
        }
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