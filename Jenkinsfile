def remote = [:]
remote.name = "ubuntu"
remote.allowAnyHosts = true
def ID
def IP
def STATE
node{
    withCredentials(
        [[
            $class: 'AmazonWebServicesCredentialsBinding',
            accessKeyVariable: 'AWS_ACCESS_KEY_ID',
            credentialsId: 'aws-client',  // ID of credentials in Jenkins
            secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
        ]]) {
        stage("create EC2 instance"){
            sh 'aws configure set region us-east-2'
            ID = sh (script: 'aws ec2 run-instances --image-id ami-05c1fa8df71875112 --count 1 --instance-type t2.micro --key-name KEY_AWS --security-group-ids sg-d77dc5b4 --subnet-id subnet-a9541dd3 --region us-east-2 --query \'Instances[0].InstanceId\'',returnStdout: true)
        }
	stage("get the EC2 external ip"){
            remote.host = sh (script: "aws ec2 describe-instances --query \'Reservations[0].Instances[0].PublicIpAddress\' --instance-ids $ID",returnStdout: true)
        }
        withCredentials([sshUserPrivateKey(credentialsId: 'KEY_AWS', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')]) {
            remote.user = userName
            remote.identityFile = identity
            stage("install dokcer.io") {
                sh 'sudo apt-get update'
                sh 'sudo apt-get install docker.io -y'
                sh 'sudo systemctl start docker'
                sh 'sudo systemctl enable docker'
            }
	    stage("pull image from docker hub"){
                withCredentials([usernamePassword(credentialsId: 'docker-acc', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
                        sh 'sudo docker pull islajd/test:firsttry'
                }
            }
	stage("run container"){
	sh 'sudo docker run -d -p 8081:8080 --name demoGrisilda1 islajd/test:firsttry'
	}
	}
    }
}
