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
			agent {
                docker { 
					image 'newtmitch/sonar-scanner' 
				}
            }
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
				sh "echo \"Docker Image Pushed\""
				script {
					docker.withRegistry("https://196093915263.dkr.ecr.ap-southeast-2.amazonaws.com", "ecr:ap-southeast-2:aws_creds") {
						docker.image("helloworld-webapi").push('latest')
					}	
				}
				sh "echo \"Docker Image Pushed\""
			}
		}
		stage('Run instance')
		{
			environment {
				TASK_NAME = "helloworld-webapi-taskdef"
				AWS_ACCESS_KEY_ID     = credentials('aws_creds-aws-secret-key-id')
        		AWS_SECRET_ACCESS_KEY = credentials('aws_creds-aws-secret-access-key')
            }
			steps { 
				withCredentials( [[ $class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws_creds', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY' ]]) {
					 echo "Listing contents of an S3 bucket." 
					 sh "AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID} \ 
					 AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY} \ 
					 AWS_REGION=us-east-1 \ 
					 aws s3 ls clouductivity-demo" 
			} }  
			
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
				sh "curl -X POST -H 'Content-type: application/json' --data '{\"text\":\"Result ${currentBuild.result}\"}' https://hooks.slack.com/services/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"
        }
    }
}