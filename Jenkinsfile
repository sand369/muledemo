pipeline {
  agent any
  stages {
    stage('Deploy Standalone') { 
      steps {
      def MVHome = tool name: 'Maven', type: 'maven'
      bat "${MVHome}/bin/mvn deploy -P standalone"
      }
    }
}
}
