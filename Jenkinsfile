pipeline {
	agent any
    stages {
        stage('Git') {
            steps { git 'https://github.com/saravananch/WebApp.git' }
        }
	stage('Build') {
	            steps { sh label: '', script: 'mvn clean'
		            sh label: '', script: 'mvn install'}
		            }

        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("saravananch/mydocker")
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-git') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
               // withCredentials([usernamePassword(credentialsId: 'GoogleCloudOsUser', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
			//withCredentials([file(credentialsId: 'GoogleCloudOsUser', variable: 'FILE')]) {
			//dir('/scratch/jenkins/') {
      			//sh 'use $FILE'
    			//}
			withCredentials(bindings: [sshUserPrivateKey(credentialsId: 'GoogleCloudOsUser', \
                              keyFileVariable: 'GoogleCloudOsUser', \
                              passphraseVariable: 'googleCloudPassPhrase', \
				usernameVariable: 'googleCloudUserName')]){
				
                    script {
                        sh "sshpass  -v ssh -o StrictHostKeyChecking=no $googleCloudUserName@$prod_ip \"docker pull saravananch/mydocker:${env.BUILD_NUMBER}\""
                        try {
                            sh "sshpass  -v ssh -o StrictHostKeyChecking=no $googleCloudUserName@$prod_ip \"docker stop mydocker\""
                            sh "sshpass -v ssh -o StrictHostKeyChecking=no $googleCloudUserName@$prod_ip \"docker rm mydocker\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass  -v ssh -o StrictHostKeyChecking=no $googleCloudUserName@$prod_ip \"docker run --restart always --name mydocker -p 9191:8080 -d saravananch/mydocker:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}
