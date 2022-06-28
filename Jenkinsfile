pipeline {
  agent any
   stages {

      stage ('Checkout SCM'){
        steps {
          checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'Github_creds', url: 'https://github.com/madhusudhan7492/Opstree-task.git']]])
        }
      }

      stage('Describe the instance'){
        steps{
            script{
                sh "aws ec2 describe-instances"

            }
            
        }
      }    
         
   } 
}