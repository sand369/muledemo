pipeline {
  agent any
  stages {
    stage('Deploy Standalone') { 
      steps {
      mvn deploy -P standalone
      }
    }
}
}
