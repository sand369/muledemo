node {

	def skipMunitTest = (params.SKIP_MUNIT_TEST != null && params.SKIP_MUNIT_TEST != false) || params.SKIP_MUNIT_TEST == true
	//def skipMunitTest = 'false'
	//boolean deployServer = (params.DEPLOYSERVER != null && params.DEPLOYSERVER != false) || params.DEPLOYSERVER == true || params.DEPLOYSERVER == null
	//def undeploy = (params.UNDEPLOY != null && params.UNDEPLOY == true)
	def undeploy = 'true'
	def startTime = new Date().format('yyyyMMddHH:mm:ss')
	//boolean deployRepository = (params.DEPLOY_REPOSITORY != null && params.DEPLOY_REPOSITORY != false) || params.DEPLOY_REPOSITORY == true || params.DEPLOY_REPOSITORY == null
        boolean deployRepository = true
	boolean deployServer = true
	
        //def MAVEN_ARTIFACTID = params.MAVEN_ARTIFACTID
        def MAVEN_ARTIFACTID = artifactid()
        echo MAVEN_ARTIFACTID
	//def git_id = "https://github.com/sand369/${MAVEN_ARTIFACTID}.git"
	def CFL_BRANCH = 'master'
	
	def MULE_ENV = params.MULE_ENV
                def ENV_PROPERTIES_LOCATION = params.ENV_PROPERTIES_LOCATION
                def VAULT_KEY = params.VAULT_KEY
                echo params.MULE_ENV
      echo params.ENV_PROPERTIES_LOCATION
                echo params.VAULT_KEY


	if (params.BRANCH != null) {
		CFL_BRANCH = params.BRANCH
	}

	def MULE_ENVIRONEMENT = params.MULE_ENVIRONEMENT
      echo params.MULE_ENVIRONEMENT
	if (params.MULE_ENVIRONEMENT != null) {
		MULE_ENVIRONEMENT = params.MULE_ENVIRONEMENT
	}

	def APP_VERSION = 'v1'
	if (params.APP_VERSION != null) {
		APP_VERSION = params.APP_VERSION
	}

	env.JAVA_HOME = "${tool 'JDK'}"
	env.PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
	echo env.JAVA_HOME
	bat 'java -version'
	
	//echo "DEBUG: parameter bar =${mule.env}"

	withMaven(jdk: 'JDK', maven: 'Maven') {

		if ("$MULE_ENVIRONEMENT" == 'uat') {
			//if (!undeploy) {

				stage('GIT FETCH')

				{
					git credentialsId: 'sand369', url: 'https://github.com/sand369/muledemo.git', branch: CFL_BRANCH

				}

				if (!skipMunitTest) {
					
					stage('MUnit Testing') {
						
						//bat "mvn clean test -U -Dmule.env=$MULE_ENV -Denv.properties.location=$ENV_PROPERTIES_LOCATION -Dvault.key=$VAULT_KEY"
						bat "mvn clean test -U"
						
					
					}

				}
				stage('Package') {
					def POM_VERSION = version()
					def INSTALL_FILE_NAME = "$MAVEN_ARTIFACTID-$POM_VERSION"
					bat 'C:/MuleSOft/apache-maven-3.5.4-bin/apache-maven-3.5.4/bin/mvn clean package -DskipMunitTests'
					archiveArtifacts allowEmptyArchive: true, artifacts: 'target/*.zip'
					archiveArtifacts allowEmptyArchive: true, artifacts: 'pom.xml'
				}

				stage('Install') {
					//Uncomment the below one if we want to added branch and build number to the jar file name
					//sh 'mvn clean install -DskipMunitTests -Dmaven.test.skip=true -DfinalName=$INSTALL_FILE_NAME'
					bat 'C:/MuleSOft/apache-maven-3.5.4-bin/apache-maven-3.5.4/bin/mvn clean install -DskipMunitTests -Dmaven.test.skip=true'
				}

				if (deployRepository) {

					stage('Deploy To Nexus'){

					//Global variables
					def NEXUS_URL = 'http://172.21.10.77:8081/repository'
					def NEXUS_REPOSITORY = 'lib-snapshot-local'
					def NEXUS_REPOSITORYID = 'nexus'
					def GROUP_ID = 'com.cfl.mule'
					def POM_VERSION = version()
					def INSTALL_FILE_NAME = "$MAVEN_ARTIFACTID"
					
				        def zip_file = "target/${INSTALL_FILE_NAME}-${POM_VERSION}.zip"
					echo """Global variables:
					Nexus URL : ${NEXUS_URL}
					Nexus Repository: ${NEXUS_REPOSITORY}
					Nexus Repository ID from Settings.xml : ${NEXUS_REPOSITORYID}
					Pom Group ID: ${GROUP_ID}
					zip file: ${zip_file}
					"""
					
						bat "C:/MuleSOft/apache-maven-3.5.4-bin/apache-maven-3.5.4/bin/mvn deploy:deploy-file -Durl=${NEXUS_URL}/${NEXUS_REPOSITORY}/ -DrepositoryId=${NEXUS_REPOSITORYID} -DgroupId=${GROUP_ID} -DartifactId=${MAVEN_ARTIFACTID} -Dversion=$POM_VERSION -Dpackaging=zip -Dfile=${zip_file} -DgeneratePom=true"
					}
					

				}
				if (deployServer) {
					stage('Deploy To Standalone') {
						//bat "C:/MuleSOft/apache-maven-3.5.4-bin/apache-maven-3.5.4/bin/mvn deploy -P standalone -Dmule.env=$MULE_ENV -Denv.properties.location=$ENV_PROPERTIES_LOCATION -Dvault.key=$VAULT_KEY"
                        bat "C:/MuleSOft/apache-maven-3.5.4-bin/apache-maven-3.5.4/bin/mvn deploy -P standalone"
							}
						}
			        
	
		} 
		else {
		stage('Copy Artifact') 
                        {
                          def PROJECT_NAME = "$MAVEN_ARTIFACTID"
      
                           echo """Global variables:
					       project-name : ${PROJECT_NAME}
					       """
      
                          copyArtifacts filter: 'target/*.zip,pom.xml', fingerprintArtifacts: true, projectName: PROJECT_NAME, selector: lastSuccessful()
                        }
                        stage('Deploy To PROD') 
                        {
bat "mvn mule:deploy -P standalone -Dmule.artifact='target//mule-demo.zip' -Dmule.applicationName='mule-demo'"
}
		
			}
		
	}
	

	}
def version() {
		def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
		matcher ? matcher[0][1] : null
	}
	def artifactid() {
		def matcher = readFile('pom.xml') =~ '<artifactId>(.+)</artifactId>'
		matcher ? matcher[0][1] : null
	}
