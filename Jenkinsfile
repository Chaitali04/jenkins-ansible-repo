pipeline {
    agent any
    
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "localhost:8081"
        NEXUS_REPOSITORY = "nexus-repo"
        NEXUS_CREDENTIAL_ID = "nexus_cred"
    }
    
    stages{
        stage('SCM'){
            steps{
                git 'https://github.com/javahometech/dockeransiblejenkins'
            }
        }
        stage('Maven Build'){
            steps{
                sh 'mvn clean package'
            }
        }
        stage("Publish to Nexus Repository Manager") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
        stage('Deploy'){
           steps{
               script{
                    def artifactUrl = "http://localhost:8081/repository/nexus-repo/in/javahome/dockeransible/1.0-SNAPSHOT/dockeransible-1.0-20220630.175647-3.war"

                    withEnv(["ARTIFACT_URL=${artifactUrl}", "APP_NAME=${pom.artifactId}"]) {
                        echo "The URL is ${env.ARTIFACT_URL} and the app name is ${env.APP_NAME}"

                        // install galaxy roles
                        // sh "ansible-galaxy install -vvv -r requirements.yml"       

                        ansiblePlaybook colorized: true,
                        credentialsId: 'ssh-jenkins',
                        // limit: "${HOST_PROVISION}",
                        installation: 'ansible',
                        inventory: 'app-server.ini',
                        playbook: 'ansible-deploy.yml',
                        sudo: true,
                        sudoUser: 'jenkins'
                    }
               }
           }
        }
    }
}

