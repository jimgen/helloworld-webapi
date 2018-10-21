#!/usr/bin/env groovy

pipeline {
	agent { label 'master' }
	stages {
		stage('Checkout') {
			steps {
				checkout scm
			}
		}
		stage('Unit Tests')
		{
			steps {
            sh "echo \"Test done\""
			//sh "dotnet test helloworld-webapi.csproj -c Release --logger \"trx;LogFileName=TestResult.xml\""
			//	sh 'cp -R TestResults/TestResult.xml .' 
			//	step([$class: 'MSTestPublisher', testResultsFile: 'TestResult.xml', failOnError: true, keepLongStdio: true])
			}
		}
		stage('Integration Tests')
		{
			steps {
            sh "echo \"Test done\""
			//sh "dotnet test helloworld-webapi.csproj -c Release --logger \"trx;LogFileName=TestResult.xml\""
			//	sh 'cp -R TestResults/TestResult.xml .' 
			//	step([$class: 'MSTestPublisher', testResultsFile: 'TestResult.xml', failOnError: true, keepLongStdio: true])
			}
		}
		stage('Code Analysis') {
			steps {
				sh "echo \"Code Analysis\""
			}
		}
		stage('Build image') {
            environment {
                IMAGE_NAME = "helloworld-webapi"
            }
			steps {
				sh "docker build -t $IMAGE_NAME ."
			}
		} 
		stage('Push image') {
			environment {
                ACCOUNT_ID = "196093915263"
				IMAGE_NAME = "helloworld-webapi"
            }
			steps {
				script {
					docker.withRegistry("https://$ACCOUNT_ID.dkr.ecr.ap-southeast-2.amazonaws.com', 'ecr:us-east-1:aws-credentials") {
						docker.image('{IMAGE_NAME}').push("v${env.BUILD_NUMBER}")
						docker.image('{IMAGE_NAME}').push('latest')
					}
					sh "echo \"Docker Image Pushed\""
				}
			}
		}
		stage('Run instance')
		{
			steps {
				sh 'sed -e "s;%BUILD_NUMBER%;${BUILD_NUMBER};g" {TASK_NAME}.json >  {TASK_NAME}-v${BUILD_NUMBER}.json'
				sh 'aws ecs register-task-definition --family {TASK_NAME} --cli-input-json file://{TASK_NAME}-v${BUILD_NUMBER}.json --region us-east-1'
				sh '''	
					TASK_REVISION=`aws ecs describe-task-definition --task-definition {TASK_NAME} --region us-east-1 | jq .taskDefinition.revision`
					SERVICES=`aws ecs describe-services --services {SERVICE_NAME} --cluster {CLUSTER_NAME} --region us-east-1 | jq '.services[] | length'`
					if [ $SERVICES == "" ]; then 
						aws ecs create-service --cluster {CLUSTER_NAME} --region us-east-1 --service {SERVICE_NAME} --task-definition {TASK_NAME}:${TASK_REVISION} --desired-count 1 
					else 
						aws ecs update-service --cluster {CLUSTER_NAME} --service {SERVICE_NAME} --task-definition {TASK_NAME}:${TASK_REVISION} --desired-count 1 --region us-east-1
					fi
				'''
			}
		}
		stage('Clean up')
		{		
			steps {
				sh 'docker image prune -a -f'
				deleteDir()
				script {
					currentBuild.result = 'SUCCESS'
				}
			}
		}
	}
	post {
        failure {
            script {
                currentBuild.result = 'FAILURE'
            }
        }

        always {
            step([$class: 'Mailer',
                notifyEveryUnstableBuild: true,
                recipients: "jimmy.gen@gmail.com",
                sendToIndividuals: true])
        }
    }
}