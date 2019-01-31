pipeline {
  agent any
  stages {
    stage('Deploy Standalone') { 
      steps {
      bat 'mvn deploy -P standalone'
      }
    }
}
}
