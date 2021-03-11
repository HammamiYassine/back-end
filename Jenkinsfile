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
        NEXUS_URL="http://localhost:8082/"
        NEXUS_REPOSITORY="backend"
        NEXUS_CREDENTIALS_ID="1234"
    }
    stages {
        stage ('SCM'){
            steps{
                git credentialsId: '1a687c74-42ea-427f-bff0-1a63654f3be1',
                url: 'https://github.com/HammamiYassine/back-end.git'
            }
        }
        stage ('Maven Build'){
            steps{
               bat "mvn clean package -Dmaven.test.skip=true -X"
            }
        }
        stage ('SonarQube analysis'){
            steps {
            withSonarQubeEnv(credentialsId: 'sonar', installationName: 'sonar') {
            bat '''
            mvn sonar:sonar'''    
            }
        }
        }
        stage ('pushing artifact to nexus'){
             steps{
               nexusArtifactUploader artifacts: [
                   [
                       artifactId: 'ForumServerSide',
                       classifier: '',
                       file: 'target/ForumServerSide-0.0.1.jar',
                       type: 'jar'
                       ]
                       ],
               credentialsId: '1234',
               groupId: 'tn.Forum',
               nexusUrl: 'localhost:8082/',
               nexusVersion: 'nexus3',
               protocol: 'http',
               repository: 'backend',
               version: '0.0.1'
             }
        }
        stage ('increment version'){
        steps {
             bat "mvn build-helper:parse-version versions:set -DnewVersion=\${parsedVersion.majorVersion}.\${parsedVersion.minorVersion}.\${parsedVersion.nextIncrementalVersion} versions:commit"
             bat "git push \"${REPO_URL}\""
            
        }
        }
        stage ('deploy'){
            steps{
        bat '''cd C:\\Users\\yassi\\.jenkins\\workspace\\back-end\\Infrastructure
copy C:\\Users\\yassi\\.jenkins\\workspace\\back-end\\target\\ForumServerSide-0.0.1-SNAPSHOT.jar .\\ForumServerSide-0.0.1-SNAPSHOT.jar
docker-compose down
docker-compose up -d '''
            }
        }
    }
}
