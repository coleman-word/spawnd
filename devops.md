# Continuous Integration & Delivery

## Install Kube Lego chart

helm install stable/kube-lego --set config.LEGO_EMAIL=<valid-email>,config.LEGO_URL=https://acme-v01.api.letsencrypt.org/directory

## Update jenkins.values.yaml

Find and replace `jenkins.acs.az.estrado.io` with the DNS name provisioned above

helm --namespace jenkins --name jenkins -f ./jenkins-values.yaml install stable/jenkins

watch kubectl get svc --namespace jenkins # wait for external ip
export JENKINS_IP=$(kubectl get svc jenkins-jenkins --namespace jenkins --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
export JENKINS_URL=http://${JENKINS_IP}:8080

kubectl get pods --namespace jenkins # wait for running
open ${JENKINS_URL}/login

printf $(kubectl get secret --namespace jenkins jenkins-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode) | pbcopy
Add credentials for private container registry (optional)

kubectl create secret docker-registry croc-hunter-secrets --docker-server=$DOCKER_SERVER --docker-username=$DOCKER_USERNAME --docker-password=$DOCKER_PASSWORD --docker-email=$DOCKER_EMAIL --namespace=croc-hunter
Reference to the secret name must also be added to the chart values.yaml or set on install.

## Login and configure Jenkins and setup pipeline

username: admin
password: <paste>

If you're not using quay you can configure this to alternate locations in Jenkinsfile.json

## Watch Jenkins build agents run

kubectl get pods --namespace jenkins
Update Org to build PRs


## Setup Webhook in Github

printf ${JENKINS_URL}/github-webhook/ | pbcopy


## Pushing Game Updates

git checkout dev
sed -i "" "s/game\.js/game2\.js/g" croc-hunter.go
git commit -am "Game 2"
git push
Building and releasing

open ${JENKINS_URL}/blue/organizations/jenkins/lachie83%2Fcroc-hunter/activity/
