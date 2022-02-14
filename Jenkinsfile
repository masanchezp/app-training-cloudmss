pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-cred-masp')
    }      

    stages {
        stage('Git Clone') {
            steps {
                echo 'Clone'
                //sh 'git clone https://github.com/masanchezp/app-training-cloudmss.git'
            }
        }
        stage('Build') {
            steps {
                echo 'Build'
                sh 'docker build --no-cache -t masprieto/app-training .'
            }
        }

        stage('Scan') {
            steps {
                // Scan the image
                prismaCloudScanImage ca: '',
                cert: '',
                dockerAddress: 'unix:///var/run/docker.sock',
                image: 'masprieto/app-training*',
                key: '',
                logLevel: 'info',
                podmanPath: '',
                project: '',
                resultsFile: 'prisma-cloud-scan-results.json',
                ignoreImageBuildTime:true
            }
        }

        stage('Login Dockerhub') {

			steps {
                echo 'Login Dockerhub'
				sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
			}
		}

		stage('Push Dockerhub') {

			steps {
                echo 'Push Dockerhub'
				sh 'docker push masprieto/app-training:latest'
			}
		}

        stage('AWS deployment') {

			steps {
                echo 'AWS Deployment'

                //withAWS(credentials: 'AWS-jenkins-credentials', region: 'us-west-2') {
                //sh 'sudo python3 -m awscli s3 ls'
                //sh 'aws s3 ls'
                sh 'sudo docker run -d --rm -it -p 80:80 masprieto/app-training'
			}
		}
    }
     post {
        always {
            // docker logout
            sh 'docker logout'
            // The post section lets you run the publish step regardless of the scan results
            prismaCloudPublish resultsFilePattern: 'prisma-cloud-scan-results.json'
        }
    }
}
