pipeline {
	agent any
	environment {
		DEPLOY_CREDS = credentials('webserver_login')
	}
	stages {
		stage('Build') {
			steps {
				echo 'Running build automation'
				sh './gradlew build --no-daemon'
				archiveArtifacts artifacts: 'dist/trainSchedule.zip'
			}
		}
		stage('DeployToStaging') {
			when {
				branch 'master'
			}
			steps {
				sh('curl -u $DEPLOY_CREDS_USR:$DEPLOY_CREDS_PSW https://4929220fd23c.mylabserver.com/') {
					sshPublisher(
						failOnError: true,
						continueOnError: false,
						publishers: [
							sshPublisherDesc(
								configName: 'staging',
								transfers: [
									sshTransfer(
										sourceFiles: 'dist/trainSchedule.zip',
										removePrefix: 'dist/',
										remoteDirectory: '/tmp',
										execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule'
									)
								]
							)
						]
					)
				}
			}
		}
		stage('DeployToProduction') {
			when {
				branch 'master'
			}
			steps {
				input 'Does the staging environment look OK?'
				milestone(1)
				sh('curl -u $DEPLOY_CREDS_USR:$DEPLOY_CREDS_PSW https://4929220fd21c.mylabserver.com/') {
					sshPublisher(
						failOnError: true,
						continueOnError: false,
						publishers: [
							sshPublisherDesc(
								configName: 'remote',
								transfers: [
									sshTransfer(
										sourceFiles: 'dist/trainSchedule.zip',
										removePrefix: 'dist/',
										remoteDirectory: '/tmp',
										execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule'
									)
								]
							)
						]
					)
				}
			}
		}
	}
}
