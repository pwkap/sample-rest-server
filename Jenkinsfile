pipeline {
   agent { docker { image 'maven:3.5-alpine' } }

   options {
      buildDiscarder(logRotator(numToKeepStr:'10'))
   }

   stages {
      stage('Build') {
         steps {
            sh 'mvn clean package'
            echo 'hello2'
         }
      }
   }
}
