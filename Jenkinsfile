pipeline {
    tools{
        jdk 'java8'
    }
    agent any
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "karthick1ktcs/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
           
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('CanaryDeploy') {
            
             environment { 
                CANARY_REPLICAS = 1
            }
            steps {
               kubeconfig(caCertificate: 'LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMvakNDQWVhZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeU1USXlNakUwTWpVeU5sb1hEVE15TVRJeE9URTBNalV5Tmxvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBT3ZYClJYN3dIZFRYNy9zNkFUVitBeGZhTXJGSlRESy9KTmZ4bmZzaW5YU01obmlyZnJ1dFlNN2dRZlh4c0pxeWdWQmMKblFUSnhHaU5Ra1BZYkptSGJOYUFEelNtNFp6aHYxVnluQnBpMHJQbFlBN1ppeDlqTHlPdTM4SGVHK2NZVy9kQQpkelpkZXl4WC84MlVnVHVDRzlBWWQzVm1TdUNSWWhaY2JYYlVMeHNQWUs2TW1hL1NEbEhvc2RRZDcxMTlQRVpiCnd5N3lZaEI4WjlaNkJNRWw1bnAwQXlnUDcxbnRjN25idDBOVXZQNUl5ZEJLUyt1aEV0dTlrV1hSMXRPcVBYdysKNDlNc0x6YzVtUGNYNHEyV1FvNnBRQTJrR1lWZkltOWVtV3UrYTZtY2o5dEYxbU84aElJdWp4U2tpaGV1R2tFRgpnTkI3K1hEZTVRRXdweDZRRWxNQ0F3RUFBYU5aTUZjd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZNWWNYbUFSY3RHOTlvZjBxQUtjeFp5N2w4RzhNQlVHQTFVZEVRUU8KTUF5Q0NtdDFZbVZ5Ym1WMFpYTXdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBTVZNTXNYbTllMzJwbmFucEkvMApIZTJsSzRVSHhWQWNlb0VUVFgyMHZDdGRnWlNic1dqczdRcnZJNU9KcnNwakJFWkN2TnlwOVgwcWtrWFBFQ2QxCkZTcVFZZHRVZ2ViMWtOSGxMUWtpWVVGUmNxcWsrSCtvUEpEbnBIMDJPWFB4V1hpUTRnaDlQVmwrUWhQV2tsemoKYlFRVFRLOHJiWFIxbTRLWGt1d0g2TnF0dWVyRFo2Z01Nelo1cmNSWUhCdUZ6aDhraUlxdHRxSmM1WDQ4cHZWVwpLUHFXWnVWYkNmY1hWMjE5cWtYdE8rSXBPVFppMkt1Z1hmRndpNHJJbnJGdFpsc00xL0pRTmZNdUtLNU02S2NpClRSYlVUSDNja092b09xeUl2Q25Ic2ZNU3JLbEVkQTIrWVJtTkNNdmNTV2lHMmlFMktOMGRqMFMySE5LQTVIU1kKMTJNPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==', credentialsId: 'kubeconfig', serverUrl: 'https://172.31.90.46:6443') {
    some block
                  //}
               // sh ('export KUBERNETES_MASTER=10.1.2.3:8080')
                //sh ('sudo su')
                //sh ('kubectl apply -f karthick1ktcs/train-schedule')
                kubernetesDeploy(
                    //kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
            }
        }
        stage('DeployToProduction') {
            
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
