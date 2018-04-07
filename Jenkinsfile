pipeline {
    agent {
        docker {
            image 'node:6-alpine' 
            args '-p 3000:3000 -p 5000:5000'
        }
    }
    //NOTE: use pypline command to set env
    environment { 
       //the location of the npm cache needs to be changed because otherwise it tries to put it into the root dir
       npm_config_cache = '/var/lib/jenkins/workspace/building-a-multibranch-pipeline-project/.npm' 
       //CI stands for cont integration and if set then the test run of node will not ask for user input.. that would hang up the pipeline
       CI = 'true'
    }
    stages {
        stage('Build') { 
            steps {
                //NOTE: this runs as expected in the new node:6-alpine container
		//      but the jenkins maps the home folder to the pipeline's home
                //      In this case the pipleine is hosted locally, not in a docker, 
                //      hance it requires permissions to run the npm. added jenkins user the correct group to solve this issue.
                sh 'npm install'
		//sh 'uname -a'
		//sh 'pwd'
		//sh 'touch created_within_docker_agent.txt'
		//sh 'ls -al /' 
            }
        }
        stage('Test') {
            steps {
                sh './jenkins/scripts/test.sh'
            }
        }
        stage('Deliver') { 
            steps {
                sh './jenkins/scripts/deliver-for-development.sh' 
                input message: 'Finished using the web site? (Click "Proceed" to continue)' 
                sh './jenkins/scripts/kill.sh' 
            }
        }
    }
}

