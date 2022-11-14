def versionPom = ""
pipeline {
    agent {
        node {
            label "nodo-java"
        }
    }
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "192.168.85.6:8081"
        NEXUS_REPOSITORY = "bootcamp"
        NEXUS_CREDENTIAL_ID = "nexus-user"
        DOCKERHUB_CREDENTIALS=credentials("lhamaoka")
        DOCKER_IMAGE_NAME="lhamaoka/spring-boot-app"
    }    
    stages{
        stage("Tests"){
            steps{
                sh "mvn test"
		jacoco()
                junit "target/surefire-reports/*.xml"
            }
        }
        stage("Builds"){
            steps{
                sh "mvn clean package -DskipTests"
            }
        }
	stage("Publish to Nexus") {
            steps {
                script {
                    // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                    pom = readMavenPom file: "pom.xml"
                    // Find built artifact under target folder
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    // Print some info from the artifact found
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    // Extract the path from the File found
                    artifactPath = filesByGlob[0].path
                    // Assign to a boolean response verifying If the artifact name exists
                    artifactExists = fileExists artifactPath                    
		   if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}"
                        versionPom = "${pom.version}"                        
		   nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId,
                                classifier: "",
                                file: artifactPath,
                                type: pom.packaging],                                // Lets upload the pom.xml file for additional information for Transitive dependencies
                                [artifactId: pom.artifactId,
                                classifier: "",
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        )                    
			} else {
                        error "*** File: ${artifactPath}, could not be found"
                    }
                }
            }
        }
    }
    stage("Build image and push to Doker Hub"){
        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
        sh "docker build -t $DOKER_IMAGE_NAME:${versionPom} ."
        sh "docker push $DOKER_IMAGE_NAME:${versionPom}"
    }
}
