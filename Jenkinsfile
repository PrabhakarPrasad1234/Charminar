pipeline {
agent any
tools {
nodejs 'NJS-11.2.0'
}
options {
quietPeriod(30)
timeout(time: 1, unit: 'HOURS')
timestamps()
}
stages {
stage('clean') {
steps {
deleteDir()
sendNotifications 'STARTED'
}
}
stage ('checkout') {
steps {
checkout([$class: 'GitSCM', branches: scm.branches, doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'source:dsa-dev1', url: 'https://source.developers.google.com/p/dsa-dev1/r/pdz-ui']]])
}
}
stage ('test') {
when {
branch 'development'
}
steps {
nodejs(nodeJSInstallationName: 'NJS-11.2.0') {
sh """
#!/bin/bash
set -e
npm install
#npm test
"""
}
}
}
stage('sonarqube') {
when {
branch 'development'
}
environment {
scannerHome = tool 'SQ'
}
steps {
withSonarQubeEnv('SQ') {
sh "${scannerHome}/bin/sonar-scanner"
}
}
}
stage('build') {
when {
branch 'development'
}
steps {
script {
app = docker.build("dsa-dev1/productizer/pdz-ui","--no-cache --build-arg foo=bar .")
}
}
}
stage('push') {
when {
branch 'development'
}
steps {
script {
docker.withRegistry('https://gcr.io', 'gcr:dsa-dev1') {
app.push("${env.BUILD_NUMBER}")
app.push("latest")
}
}

}
}
stage('security') {
when {
branch 'security'
}
steps {
sh """
echo "gcr.io/dsa-dev1/productizer/pdz-ui:${env.BUILD_NUMBER} `pwd`/Dockerfile" > anchore_images
"""
anchore name: 'anchore_images'
}
}
stage('deploy-dev'){
when {
branch 'development'
}
steps {
script {
sh """
#!/bin/bash
scp -r -o StrictHostKeyChecking=no -i ~/.ssh/id_ed25519 deploy/* config/* pdzuser@mypdz.dev.dsa.ai:~/deploy/ui
ssh -o StrictHostKeyChecking=no -v -i ~/.ssh/id_ed25519 pdzuser@mypdz.dev.dsa.ai "chmod +x ~/deploy/ui/deploy-ui.sh"
ssh -o StrictHostKeyChecking=no -v -i ~/.ssh/id_ed25519 pdzuser@mypdz.dev.dsa.ai "cd ~/deploy/ui; ./deploy-ui.sh development ${env.BUILD_NUMBER}"
sleep 30s
"""
}
}
}
stage ('smoketest-dev') {
when {
branch 'development'
}
steps {
sh '''
#!/bin/bash
set -eo pipefail
status=$(curl -L -s -o /dev/null -w "%{http_code}" http://mypdz.dev.dsa.ai)
if [[ "$status" -eq "200" ]]; then
exit 0
else
echo "Smoke test failed."
exit 1
fi
'''
}
}
stage('teardown') {
when {
branch 'security'
}
steps {
sh'''
for i in `cat anchore_images | awk '{print $1}'`;do docker rmi $i; done
'''
}
}
}
post {
always {
sendNotifications currentBuild.result
}
}
}

