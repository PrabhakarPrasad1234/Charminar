pipeline {
	agent any

	stages {
		stage('Prep') {

			steps {
				deleteDir()
				git 'https://github.com/PrabhakarPrasad1234/Charminar.git'
				sh "python -version"
			}
		}
		stage('Build') {

			steps {
				echo "charminar...Build"
			}
		}
		stage('Code Quality') {

			steps {
				echo "charminar...QQ Sonar"
			}
		}
		stage('Package') {

			steps {
				echo "charminar...Package"
			}
		}
		stage('Publish') {

			steps {
				echo "charminar...Publish via Anchore scan"
			}
		}
		stage('Notify') {

			steps {
				echo "charminar...Notify via Flowdock"
			}
		}

	}
}
