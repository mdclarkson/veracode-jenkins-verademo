pipeline {
    agent any

    environment {
        VERACODE_APP_NAME = 'Verademo'      // App Name in the Veracode Platform
    }

    // this is optional on Linux, if jenkins does not have access to your locally installed docker
    //tools {
        // these match up with 'Manage Jenkins -> Global Tool Config'
        //'org.jenkinsci.plugins.docker.commons.tools.DockerTool' 'docker-latest' 
    //}

    options {
        // only keep the last x build logs and artifacts (for space saving)
        buildDiscarder(logRotator(numToKeepStr: '20', artifactNumToKeepStr: '20'))
    }

    stages {
        stage('Build') {
            steps {
                // Compile Java app
                sh 'mvn clean package'
                // pull docker container
                //sh 'doker pull juliantotzek/verademo1-tomcat'
            }
        }

        stage('Security Scan Master Branch') {
            when {
                expression {env.GIT_BRANCH == 'master'}    
            }
            steps {
                        withCredentials([usernamePassword(credentialsId: 'VeracodeAPI', passwordVariable: 'VERACODEKEY', usernameVariable: 'VERACODEID')]) {
                            veracode applicationName: "Jenkins_Verademo_Java_MultiStage", canFailJob: true, timeout: 60, criticality: 'VeryHigh',
                            fileNamePattern: '', replacementPattern: '', scanExcludesPattern: '', scanIncludesPattern: '',
                            scanName: 'build $buildnumber - Jenkins',
                            uploadExcludesPattern: '', uploadIncludesPattern: 'target/*.war', waitForScan: true,
                            vid: VERACODEID, vkey: VERACODEKEY
               
                        }
                    }

        stage ('Veracode SCA') {
            steps {
                echo 'Veracode SCA'
               withCredentials([string(credentialsId: 'SRCCLR_API_TOKEN', variable: 'SRCCLR_API_TOKEN')]) {
                            sh 'curl -sSL https://download.sourceclear.com/ci.sh | sh'

                                // debug, no upload
                                //sh "curl -sSL https://download.sourceclear.com/ci.sh | DEBUG=1 sh -s -- scan --no-upload"
                            }
                           

        // only works on *nix, as we're building a Linux image
        //  uses the natively installed docker
        stage ('Deploy') {
            when { expression { return (isUnix() == true) } }
            steps {
                echo 'building Docker image'
                sh 'docker version'

                ansiColor('xterm') {
                    sh 'docker build -t verademo:${BUILD_TAG} .'
                }
                
                // split into separate stage??
                echo 'Deploying ...'
        
            }
        }
    }
}
}
}
}
