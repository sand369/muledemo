pipeline {
  agent any
  stages {
    stage('Deploy Standalone') { 
      steps {
      bash 'mvn deploy -P standalone'
      }
    }
}
}
