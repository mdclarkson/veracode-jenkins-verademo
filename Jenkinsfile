pipeline {
    agent {
        docker {
            image 'maven:3.6-jdk-11'
            args '-v /root/.m2:/root/.m2'
        }
    }
    options {
        skipStagesAfterUnstable()
    }
    environment {
        JAVA_OPTS="-Duser.home=${WORKSPACE}"
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
            }
            steps {
                parallel(
                    a:{
                         //Policy scan
                        withCredentials([usernamePassword(credentialsId: 'VeracodeAPI', passwordVariable: 'VERACODEKEY', usernameVariable: 'VERACODEID')]) {
                            veracode applicationName: "Jenkins_Verademo_Java_MultiStage", canFailJob: true, timeout: 60, criticality: 'VeryHigh',
                            fileNamePattern: '', replacementPattern: '', scanExcludesPattern: '', scanIncludesPattern: '',
                            scanName: 'build $buildnumber - Jenkins',
                            uploadExcludesPattern: '', uploadIncludesPattern: 'target/*.war', waitForScan: true,
                            vid: VERACODEID, vkey: VERACODEKEY
               
                        }
                    },
                    b:{
                        // 3rd party scan application
                        //catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE')
                        withCredentials([string(credentialsId: 'SRCCLR_API_TOKEN', variable: 'SRCCLR_API_TOKEN')]) {
                            sh 'curl -sSL https://download.sourceclear.com/ci.sh | sh'
                        }
                    },
                )
            }
        }
    
      

    }
