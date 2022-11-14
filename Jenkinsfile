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
            }
        }
        stage("Build"){
            steps{
                sh "mvn clean package -DskipTests"
            }
        }
    }
}
