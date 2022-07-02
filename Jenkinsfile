@Library('github.com/releaseworks/jenkinslib') _
pipeline {
  agent any

    //environment varible 
  environment {
        OLD_INSTANCE_TYPE = 't2.micro'
    }

  stages {
    // Clone the github repo on jenkins workspace
    stage('Checkout SCM') {
      steps {
        checkout([$class: 'GitSCM', branches: [
          [name: '*/master']
        ], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [
          [credentialsId: 'Github_creds', url: 'https://github.com/madhusudhan7492/Opstree-task.git']
        ]])
      }
    }

    //Describe the instance status and stop the instance
    stage("Describe and stop the instance") {
      steps {
        withCredentials([
          [$class: 'UsernamePasswordMultiBinding', credentialsId: 'aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']
        ]) {
          AWS("--region=us-east-1 ec2 describe-instances --query 'Reservations[*].Instances[*].{PublicIP:PublicIpAddress,Name:Tags[?Key=='Name']|[0].Value,Status:State.Name,InstanceID:InstanceId}' --filters Name=instance-state-name,Values=running --output table")
          AWS("--region=us-east-1 ec2 stop-instances --instance-ids $Instance_Id")
        }
        sh "echo 'Wait for 25 seconds to let the instance turn into stop state'"
        sh "sleep 25"       
        
      }
    }
    //change the instance type as per the input parameter given before running the job
    stage("Change the instance type") {
      steps {
        withCredentials([
          [$class: 'UsernamePasswordMultiBinding', credentialsId: 'aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']
        ]) {
          AWS("--region=us-east-1 ec2 describe-instances --instance-ids $Instance_Id --query 'Reservations[*].Instances[*].{PublicIP: PublicIpAddress, Name:Tags[?Key== 'Name']|[0].Value,Status:State.Name,InstanceID:InstanceId,Instancetype:InstanceType}' --output table")  
          AWS("--region=us-east-1 ec2 modify-instance-attribute --instance-id $Instance_Id --instance-type '{\"Value\": \"$Instance_Type\"}'")
          AWS("--region=us-east-1 ec2 describe-instances --instance-ids $Instance_Id --query 'Reservations[*].Instances[*].{PublicIP: PublicIpAddress, Name:Tags[?Key== 'Name']|[0].Value,Status:State.Name,InstanceID:InstanceId,Instancetype:InstanceType}' --output table")
        }
      }
    }

    //Start the instance (testing_server)
    stage("Start the instance") {
        steps {
        withCredentials([
          [$class: 'UsernamePasswordMultiBinding', credentialsId: 'aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']
        ]) {
          AWS("--region=us-east-1 ec2 start-instances --instance-ids $Instance_Id")
          sh "sleep 25"
          AWS("--region=us-east-1 ec2 describe-instances --instance-ids $Instance_Id --query 'Reservations[*].Instances[*].{PublicIP: PublicIpAddress, Name:Tags[?Key== 'Name']|[0].Value,Status:State.Name,InstanceID:InstanceId,Instancetype:InstanceType}' --output table")
        }

      }
    }
    //Run ansible playbook on the testing_server
    stage('run ansible playbook') {
      steps {
        //here main.yml file is in the cloned repository
        ansiblePlaybook credentialsId: 'ansadmin', disableHostKeyChecking: true, installation: 'ansible', playbook: 'main.yml'
      }
    }

    //stop the instance the change the instance type to t2.micro
    stage("Stop the instance and flip back to previous instance type and start the instance") {
      steps {
        withCredentials([
          [$class: 'UsernamePasswordMultiBinding', credentialsId: 'aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']
        ]) {
          AWS("--region=us-east-1 ec2 stop-instances --instance-ids $Instance_Id")
          sh "sleep 25"
          AWS("--region=us-east-1 ec2 describe-instances --instance-ids $Instance_Id --query 'Reservations[*].Instances[*].{PublicIP: PublicIpAddress, Name:Tags[?Key== 'Name']|[0].Value,Status:State.Name,InstanceID:InstanceId,Instancetype:InstanceType}' --output table")  
          AWS("--region=us-east-1 ec2 modify-instance-attribute --instance-id $Instance_Id --instance-type '{\"Value\": \"${OLD_INSTANCE_TYPE}\"}'")
          AWS("--region=us-east-1 ec2 start-instances --instance-ids $Instance_Id")
          sh "sleep 25"
          AWS("--region=us-east-1 ec2 describe-instances --instance-ids $Instance_Id --query 'Reservations[*].Instances[*].{PublicIP: PublicIpAddress, Name:Tags[?Key== 'Name']|[0].Value,Status:State.Name,InstanceID:InstanceId,Instancetype:InstanceType}' --output table")
        }
      }
    }
    

  }
  // post{
  //       always{
  //           cleanWs()
  //       }
    }
}