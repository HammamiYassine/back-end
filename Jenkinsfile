pipeline{
    agent any
    tools {
    jdk "JAVA"    
    maven 'maven'
    }
    environment {
        REPO_URL= 'https://github.com/HammamiYassine/back-end.git'
        NEXUS_VERSION= "3"
        NEXUS_PROTOCOL="http"
        NEXUS_URL="http://192.168.0.106:8082/"
        NEXUS_REPOSITORY="back"
        NEXUS_CREDENTIALS_ID="nexus"
    }
    stages {
        stage ('SCM'){
            steps{
                git credentialsId: '1a687c74-42ea-427f-bff0-1a63654f3be1',
                url: 'https://github.com/HammamiYassine/back-end.git'
            }
        }
        stage ('remove snapshot') {
            steps {
             sh 'mvn versions:set -DremoveSnapshot'
        }
        }
        stage ('Maven Build'){
            steps{
               sh "mvn clean package -Dmaven.test.skip=true -X"
            }
        }
        stage ('SonarQube analysis'){
            steps {
            withSonarQubeEnv(credentialsId: 'sonar', installationName: 'sonar') {
            sh '''
            mvn sonar:sonar'''    
            }
        }
        }
        stage ('Quality gate') {
            steps {
                script {
           timeout(time: 1, unit: 'HOURS') { 
           def qg = waitForQualityGate() 
           if (qg.status != 'OK') {
             error "Pipeline aborted due to quality gate failure: ${qg.status}"
                                  }
                                            }
                }
            }
        }
        stage ('pushing artifact to nexus'){
             steps{
                 script {
                     def POM = readMavenPom file: 'pom.xml'
                     def VERSION = POM.version
               nexusArtifactUploader artifacts: [
                   [
                       artifactId: 'ForumServerSide',
                       classifier: '',
                       file: "target/ForumServerSide-${VERSION}.jar",
                       type: 'jar'
                       ]
                       ],
               credentialsId: 'nexus',
               groupId: 'tn.Forum',
               nexusUrl: '192.168.0.106:8082/',
               nexusVersion: 'nexus3',
               protocol: 'http',
               repository: 'back',
               version: "${VERSION}"
             }
             }
        }
        stage('Increment Version') {
            steps {
                script {
                    sh 'git config --global user.email "yassine_hammamii@yahoo.com"'
                    sh 'git config --global user.name "HammamiYassine"'
                    sh 'mvn release:update-versions'
                    sh 'git add .'
                    sh 'git commit -m "change commit"'
                    sh 'git remote set-url origin git@github.com:HammamiYassine/back-end.git'
                    sh 'git push -u origin master'
                }  
                }
    }
}
}
