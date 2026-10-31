pipline {
	agent any


	options {
		skipDefaultCheckout()
		timestamps()
		buildDiscarder(logRotator(numToKeepStr: '10'))
	}

	triggers {
		pollSCM('H/2 * * * *')
	}

	environment {
		IMAGE = 'mmoml/devops-lab'
	}


	stage {
		stage('Checkout') {
			steps {
				checkout scm
				sh 'git log --online -3'
			}
		}
		
		stage('Проверка') {
			steps {
				ansiblePlaybook(
					playbook: 'site.yml', 
					inventory: 'inventory.ini,
					installation: 'ansible',
					extras: '--syntax-check'
				)
				ansiblePlaybook(
					playbook: 'app.yml',
					inventory: 'inventory.ini',
					installation: 'ansiblr',
					extras: '--syntax-check'
				)
			}
		}

	stage('Развёртывание') {
		steps {
			ansiblePlaybook(
				playbook: 'site.yml',
				inventory: 'inventory.ini',
				installation: 'ansible',
				credentialsId: 'ansible-ssh',
				disableHostKeyChecking: true
			)
		}
	}
	stage('Вход в Docker Hub') {
		steps {
			withCedentials([usernamePassword(credentialsId: 'dockerhub',
							usernameVariable: 'DOCKERHUB_USER',
							passwordVariable: 'DOCKERHUB_TOKEN')]) {
												sh 'echo "$DOCKERHUB_TOKEN" | docker login -u "$DOCKERHUB_USER" --password-stdin'
											}
		}

	}

	stage('Публикация') {
		steps {
			sh 'docker push $IMAGE:$BUILD_NUMBER'
			sh 'docker push $IMAGE:latest'
		}
	}
	
	stage('Запуск') {
		steps {
			ansiblePlaybook(
				playbook: 'app.yml',
				inventory: 'inventory.ini',
				installation: 'ansible',
				credentials: 'ansible-ssh',
				disableHostKeyChecking: true,
				extraVars: [app_image: env.IMAGE, app_tag: env.BUILD_NUMBER]
			)
		}
	}

	post {
		always {
			sh 'docker logout'
		}
		success {
			echo "Сборка ${env.BUILD_NUMBER}: образ ${env.IMAGE}:${env.BUILD_NUMBER} запущен на узлах"
		}

		failure {
			echo "Сборка ${env.BUILD_NUMBER} упала: смотрите Console Output"
		}
	}
}
