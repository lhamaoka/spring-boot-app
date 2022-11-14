pipeline {
    agent {
        node {
            label "nodo-java"
        }
    }    
    stages{
        stage("Tests"){
            steps{
                sh "mvn test"
                junit "target/surefire-reports/*.xml"
            }
        }
        stage("Build"){
            steps{
                sh "mvn clean package -DskipTests"
            }
        }
    }
}
