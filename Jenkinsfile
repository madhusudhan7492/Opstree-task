@Library('github.com/releaseworks/jenkinslib') _
pipeline {
  agent any

  stages {

    stage('Checkout SCM') {
      steps {
        checkout([$class: 'GitSCM', branches: [
          [name: '*/master']
        ], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [
          [credentialsId: 'Github_creds', url: 'https://github.com/madhusudhan7492/Opstree-task.git']
        ]])
      }
    }

    stage("Describe and stop the instance") {
      steps {
        withCredentials([
          [$class: 'UsernamePasswordMultiBinding', credentialsId: 'aws-key', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']
        ]) {
          AWS("--region=us-east-1 ec2 describe-instances --query 'Reservations[*].Instances[*].{PublicIP:PublicIpAddress,Name:Tags[?Key=='Name']|[0].Value,Status:State.Name,InstanceID:InstanceId}' --filters Name=instance-state-name,Values=running --output table")
          AWS("--region=us-east-1 ec2 stop-instances --instance-ids $Instance_Id")
        }
        sh "echo 'Wait for 25 seconds to let the instance turn into stop state'"
        sh "sleep 25"
        sh "echo 'After stopping the instance'" 
        AWS("--region=us-east-1 ec2 describe-instances --instance-ids $Instance_Id --query 'Reservations[*].Instances[*].{PublicIP: PublicIpAddress, Name:Tags[?Key== 'Name']|[0].Value,Status:State.Name,InstanceID:InstanceId,Instancetype:InstanceType}' --output table")
        
      }
    }

    stage("Change the instance type") {
      steps {
        withCredentials([
          [$class: 'UsernamePasswordMultiBinding', credentialsId: 'aws-key', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']
        ]) {
          AWS("--region=us-east-1 ec2 modify-instance-attribute --instance-id $Instance_Id --instance-type '{\"Value\": \"$Instance_Type\"}'")
          AWS("--region=us-east-1 ec2 describe-instances --instance-ids $Instance_Id --query 'Reservations[*].Instances[*].{PublicIP: PublicIpAddress, Name:Tags[?Key== 'Name']|[0].Value,Status:State.Name,InstanceID:InstanceId,Instancetype:InstanceType}' --output table")
        }
      }
    }

    stage("Start the instance") {
      steps {
        withCredentials([
          [$class: 'UsernamePasswordMultiBinding', credentialsId: 'aws-key', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']
        ]) {
          AWS("--region=us-east-1 ec2 start-instances --instance-ids $Instance_Id")
          sh "sleep 25"
          AWS("--region=us-east-1 ec2 describe-instances --instance-ids $Instance_Id --query 'Reservations[*].Instances[*].{PublicIP: PublicIpAddress, Name:Tags[?Key== 'Name']|[0].Value,Status:State.Name,InstanceID:InstanceId,Instancetype:InstanceType}' --output table")
        }

      }
    }

    stage('run ansible playbook') {
      steps {
        ansiblePlaybook credentialsId: 'ansadmin', disableHostKeyChecking: true, installation: 'ansible', playbook: 'main.yml'
      }

    }

  }
}