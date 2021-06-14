pipeline {

environment {
 	registry = "zarczynsky/lab09"
 	registryCredential = "dockerhub"
 	dockerImage = ""
 }

agent any
	
stages {
	 stage('Build') {
	  steps {
	   echo 'Building.'
		sh 'eval "$(docker-machine env default)"'
	        sh 'git pull origin master'
		sh 'curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose'
		sh 'chmod +x /usr/local/bin/docker-compose'
		//sh "docker-compose up -d"
		sh 'cd client'
		script {
 			dockerImage = docker.build registry ./client
 		}
	}
	   post {
		failure {
		   	sendEmailAfter('Build failed')
        	}
       	 	success {
           		sendEmailAfter('Build successful')
       	 }
   	    }
	}
	  stage('Test') {
	   steps {
	   	sh '-c './src/wait.sh app_server:8081 -- echo READY && node ./src/client.js''
	   }
	   post {
        	 failure {
          		sendEmailAfter('Tests failed')
        	}
        	success {
            		sendEmailAfter('Tests successful')
       	 }
   	    }
	}
	stage('Deploy'){
		steps{
			script {
			docker.withRegistry( "", registryCredential ) {
 			dockerImage.push()
 			dockerImage.push("latest")}
		}
	}
    }
}
}

def sendEmailAfter(status){
 	echo status
            emailext attachLog: true,
                body: status,
                recipientProviders: [developers(), requestor()],
                to: 'adamegnon@gmail.com',
                subject: "Jenkins stage status"
}
