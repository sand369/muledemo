pipeline {
  agent any
  stages {
    stage('Deploy Standalone') { 
      steps {
      bat 'C:/MuleSOft/apache-maven-3.5.4-bin/apache-maven-3.5.4/bin/mvn -X deploy -P standalone'
      }
    }
}
}
