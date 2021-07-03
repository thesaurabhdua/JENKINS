def msTeamsWebhook="https://outlook.office.com/webhook/cd1ed0fa-bbe1-403c-be3d-9a6ada5c730c@f03022ed-34ea-4c8f-91cb-9abee0a20907/JenkinsCI/50b2fb4a669544028314e515700ae187/44eb6c0b-2278-48c9-9c70-198058061cfe"
def privateIP

pipeline {
 agent {
   node {
       label 'window-slave2'
   }
 }
 options {
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
        timestamps()
        timeout(time: 120, unit: 'MINUTES')
 }
 parameters {
 string(name: 'sourceBranch', description: 'Enter the source branch name' , defaultValue: 'A2019-PerformanceSuite')
 string(name: 'iqbotBranchName', description: 'Enter the IQBot branch name' , defaultValue: 'A2019')

 }
 environment {
        AWS_ACCESS_KEY = credentials('A2019_S3_Access_Key')
        AWS_SECRET_KEY = credentials('A2019_S3_Secret_Key')
        AWS_REGION = credentials('A2019_Default_Region')
        CR_AWS_SECRET_KEY = credentials('CR_AWS_SECRET_KEY')
        CR_AWS_ACCESS_KEY = credentials('CR_AWS_ACCESS_KEY')
        JENKINS_SECRET = credentials('A2019_Jenkins_Node_Secret')

 }
stages {
        stage("Pre-requisites to Launch_Instance") {
            steps {
                script {
                    set_env_variables()
                    print_env_variables()

                }
                deleteDir()
                checkout_SCM_A2019('a2019')
                aws_user_config("terraform")
                script{
                    privateIP = terraform_init()
                    println("privateIP = ${privateIP}")
                }

              }
          }

          stage('Initialize Jenkins') {
             steps {
               script{
                   bat "cmdkey /generic:TERMSRV/${privateIP} /user:Administrator /pass:VM3u-Qsb$Utevog;Y3hBk3&)KWlB*%Qj && mstsc /v:${privateIP} /f "
                   powershell """ Start-Sleep -s 180 """


               }
                 }

         }
       }
     }

     def set_env_variables(){
       env.MASTER_NODE = "${env.NODE_NAME}"
       env.MASTER_WS = "${env.WORKSPACE}"
       env.A2019_WS ="C:\\Jenkins\\workspace\\A2019_NightlyBuildRun_Performance_Suite"
       env.LICENSE_PATH = "C:\\CR-Licenses"
       env.CRLicenseS3 = "s3://aa.controlroom.snapshot/Licences/hopper/?region=us-west-2"
     }

     def print_env_variables(){
         println "INSTANCE_TYPE = ${env.INSTANCE_TYPE}"
         println "Node = ${env.MASTER_NODE}"
         println "Workspace = ${env.MASTER_WS}"
         println "Jenkins Secret = ${JENKINS_SECRET}"
     }

     def checkout_SCM_A2019(repo) {
        switch(repo) {
           case "a2019" :
              checkout([$class: 'GitSCM', branches: [[name: "${params.sourceBranch}"]], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'GitLFSPull'],[$class: 'LocalBranch', localBranch: '**']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'ba3c36be-cab8-45ae-9d05-c4871235c073', url: 'git@bitbucket.org:automationanywhere/cognitive.git']]])
              break
        }
     }

     def aws_user_config(PROFILE) {
       script {
          switch(PROFILE) {

             case "terraform" :
                   powershell '''  aws configure set aws_access_key_id $env:AWS_ACCESS_KEY --profile a2019-terraform'''
                   powershell '''  aws configure set aws_secret_access_key $env:AWS_SECRET_KEY --profile a2019-terraform'''
                   powershell '''  aws configure set region $env:AWS_REGION --profile a2019-terraform'''
                   break

             case "CR" :
                   powershell '''  aws configure set aws_access_key_id $env:CR_AWS_ACCESS_KEY --profile cr'''
                   powershell '''  aws configure set aws_secret_access_key $env:CR_AWS_SECRET_KEY --profile cr'''
                   powershell '''  aws configure set region $env:AWS_REGION --profile cr'''
                   break

             case "IQBOT" :
                   powershell ''' aws configure set aws_access_key_id AKIAWTF63LC5L4VWEIBF  --profile iqbot  '''
                   powershell ''' aws configure set aws_secret_access_key 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 --profile iqbot  '''
                   powershell ''' aws configure set region us-west-1 --profile iqbot  '''
                   break

             case "License" :
                   powershell ''' aws configure set aws_access_key_id AKIAWTF63LC5L4VWEIBF  --profile license  '''
                   powershell ''' aws configure set aws_secret_access_key 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 --profile license  '''
                   powershell ''' aws configure set region us-west-2 --profile license  '''
                   break

          }
       }
     }

     def terraform_init(){
        dir("${env.MASTER_WS}\\performance\\terraform\\onDemand") {
            powershell """
                 aws s3 cp s3://A2019-PerformanceSuite . --recursive --profile a2019-terraform --no-progress
                 """
              powershell """
               .\\terraform.exe init
                 """

             powershell """
               .\\terraform.exe plan
                 """
             powershell """
               .\\terraform.exe apply -auto-approve
                 """
             powershell """ Start-Sleep -s 300 """
             powershell """ .\\terraform.exe output private_ip """
             privateIP = powershell(script: ".\\terraform.exe output private_ip", returnStdout: true).trim() as String
             println("${privateIP}")
             return privateIP
        }
     }

     def terraform_destroy() {
       dir("${env.MASTER_WS}\\performance\\terraform\\onDemand") {
            powershell """
                 aws s3 cp s3://A2019-PerformanceSuite . --recursive --profile a2019-terraform --no-progress
                 """
              powershell """
               .\\terraform.exe init
                 """

             powershell """
               .\\terraform.exe plan
                 """
             powershell """
               .\\terraform.exe destroy -auto-approve
                 """
             powershell """ Start-Sleep -s 60 """
        }
     }
