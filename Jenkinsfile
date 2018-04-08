pipeline {
   //NOTE: use pypline command to set env
    environment {
        //devlare the dev and prod ports in one place so that they can be easily changed/maintined
        aaDEV_PORT = '3300'
        aaPROD_PORT = '5500'

       //the location of the npm cache needs to be changed because otherwise it tries to put it into the root dir
       //but in case of multi branch builds the name of the working folder is not easy to guess... now trying to use the ./ to use the initial work dir which supposed to be the working dir.
       //NOTE: actually the workdir is created with some random unique id appeneded to it, but the same time the dir length is limited to 81 or so.. so it ends up chupping off the
       //      first charactes of the project name maing it difficult to recognize the folder name... but the good thing is that the initial dir is the working dir, so the ./ works.
       npm_config_cache = './.npm'
       //CI stands for cont integration and if set then the test run of node will not ask for user input.. that would hang up the pipeline
       CI = 'true'
    }

    agent {
        docker {
            image 'node:6-alpine' 
            args '-p ${env.aaDEV_PORT}:${env.aaDEV_PORT} -p ${env.aaPROD_PORT}:${env.aaPROD_PORT}'
        }
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
        stage('Deliver for Dev') { 
            when {
                anyOf {
                     branch 'dev'
                     branch 'qa'
                }
            }
            environment {
                //try to set the dev and qa port to something different then the default 3000 port
                PORT = '${env.aaDEV_PORT}'
            }
            steps {
                sh './jenkins/scripts/deliver-for-development.sh' 
                input message: 'Finished using the web site? (Click "Proceed" to continue)' 
                sh './jenkins/scripts/kill.sh' 
            }
        }
        stage('Deploy for production') {
            when {
                branch 'prod'
            }
            environment {
                //try to set the production port to something different then the serve.js's default 5000 port (whihc is set in /node_modules/serve/lib/options.js)
                PORT = '${env.aaPROD_PORT}'
            }
            steps {
                sh './jenkins/scripts/deploy-for-production.sh'
                input message: 'Finished using the web site? (Click "Proceed" to continue)'
                sh './jenkins/scripts/kill.sh'
            }
        }
    }
}

