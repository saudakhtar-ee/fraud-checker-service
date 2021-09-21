pipeline {
	agent any
	environment {
		TEAM_NAME = "replace-saud"
		AWS_REGION = "us-east-2"
		SHORT_COMMIT_ID = "${GIT_COMMIT}".substring(0, 7)
		SERVICE_NAME = "fraud-checker-service"
		ECR_REPOSITORY_NAME = "${TEAM_NAME}-bootcamp-2021-ecr/${SERVICE_NAME}"
		ECR_REPOSITORY_FULL_NAME = "038062473746.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY_NAME}"
		ECR_IMAGE_ID = "${ECR_REPOSITORY_FULL_NAME}:${SHORT_COMMIT_ID}"
	}
	stages {
		stage('Build & Test') {
			steps {
				sh './gradlew clean build'
			}
		}
		stage('Build docker image') {
			steps {
				sh '''
					COMMIT_ID=$(git rev-parse HEAD)
					docker build -t fraud-checker-service:${SHORT_COMMIT_ID} .
				'''
			}
		}
		stage('Create ECR repo if doesnt exists') {
			steps {
				sh '''
					aws ecr create-repository \\
						--repository-name ${ECR_REPOSITORY_NAME} \\
						--image-scanning-configuration scanOnPush=true --region ${AWS_REGION} || true
				'''
			}
		}
		stage('Push docker image to ECR') {
			steps {
				sh '''
					eval $(aws ecr get-login --no-include-email --region ${AWS_REGION})
					docker tag ${SERVICE_NAME}:${SHORT_COMMIT_ID} ${ECR_REPOSITORY_FULL_NAME}:${SHORT_COMMIT_ID}
					docker tag ${SERVICE_NAME}:${SHORT_COMMIT_ID} ${ECR_REPOSITORY_FULL_NAME}:latest
					docker push ${ECR_REPOSITORY_FULL_NAME}:${SHORT_COMMIT_ID}
					docker push ${ECR_REPOSITORY_FULL_NAME}:latest
				'''
			}
		}
	}
}