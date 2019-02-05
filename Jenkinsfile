node {

	def skipMunitTest = (params.SKIP_MUNIT_TEST != null && params.SKIP_MUNIT_TEST != false) || params.SKIP_MUNIT_TEST == true
	//def skipMunitTest = 'false'
	boolean deployServer = (params.DEPLOYSERVER != null && params.DEPLOYSERVER != false) || params.DEPLOYSERVER == true || params.DEPLOYSERVER == null
	//def undeploy = (params.UNDEPLOY != null && params.UNDEPLOY == true)
	def undeploy = 'true'
	def startTime = new Date().format('yyyyMMddHH:mm:ss')
	//boolean deployRepository = (params.DEPLOY_REPOSITORY != null && params.DEPLOY_REPOSITORY != false) || params.DEPLOY_REPOSITORY == true || params.DEPLOY_REPOSITORY == null
        boolean deployRepository = true
        def MAVEN_ARTIFACTID = 'mule-demo'
	def CFL_BRANCH = 'master'

	if (params.BRANCH != null) {
		CFL_BRANCH = params.BRANCH
	}

	def MULE_ENVIRONEMENT = 'TEST'

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

	withMaven(jdk: 'JDK', maven: 'Maven') {

		if ("$MULE_ENVIRONEMENT" == 'TEST') {
			//if (!undeploy) {

				stage('GIT FETCH')

				{
					git credentialsId: 'sand369', url: 'https://github.com/sand369/muledemo.git', branch: CFL_BRANCH

				}

				if (!skipMunitTest) {

					stage('MUnit Testing') {
						bat "C:/MuleSOft/apache-maven-3.5.4-bin/apache-maven-3.5.4/bin/mvn clean test -U"
					}

				}
				stage('Package') {
					def POM_VERSION = '1.0.0-SNAPSHOT'
					def INSTALL_FILE_NAME = "$MAVEN_ARTIFACTID-$POM_VERSION"
					bat 'C:/MuleSOft/apache-maven-3.5.4-bin/apache-maven-3.5.4/bin/mvn clean package'
					archiveArtifacts allowEmptyArchive: true, artifacts: 'target/*.zip'
					archiveArtifacts allowEmptyArchive: true, artifacts: 'pom.xml'
				}

				stage('Install') {
					//Uncomment the below one if we want to added branch and build number to the jar file name
					//sh 'mvn clean install -DskipMunitTests -Dmaven.test.skip=true -DfinalName=$INSTALL_FILE_NAME'
					bat 'C:/MuleSOft/apache-maven-3.5.4-bin/apache-maven-3.5.4/bin/mvn clean install -DskipMunitTests -Dmaven.test.skip=true'
				}

				if (deployRepository) {

					stage('Deploy To Nexus')

					//Global variables
					def NEXUS_URL = 'http://172.21.10.77:8081/repository'
					def NEXUS_REPOSITORY = 'lib-snapshot-local'
					def NEXUS_REPOSITORYID = 'central'
					def GROUP_ID = 'com.cfl.mule'
					def POM_VERSION = '1.0.0-SNAPSHOT'
					def INSTALL_FILE_NAME = "$MAVEN_ARTIFACTID"
					zip zip_file = "target/$INSTALL_FILE_NAME-$POM_VERSION.zip"

					echo """Global variables:
					Nexus URL : ${NEXUS_URL}
					Nexus Repository: ${NEXUS_REPOSITORY}
					Nexus Repository ID from Settings.xml : ${NEXUS_REPOSITORYID}
					Pom Group ID: ${GROUP_ID}
					"""
					if ("$MAVEN_ARTIFACTID" == 'lib-parent-pom')

					{
						bat "C:/MuleSOft/apache-maven-3.5.4-bin/apache-maven-3.5.4/bin/mvn deploy:deploy-file -Durl=${NEXUS_URL}/${NEXUS_REPOSITORY}/ -DrepositoryId=${NEXUS_REPOSITORYID} -DgroupId=${GROUP_ID} -DartifactId='lib-parent-pom' -Dversion=$POM_VERSION -Dpackaging=pom -Dfile=target/$INSTALL_FILE_NAME.pom -DgeneratePom=true"
					} else {
						bat "C:/MuleSOft/apache-maven-3.5.4-bin/apache-maven-3.5.4/bin/mvn deploy:deploy-file -Durl=${NEXUS_URL}/${NEXUS_REPOSITORY}/ -DrepositoryId=${NEXUS_REPOSITORYID} -DgroupId=${GROUP_ID} -DartifactId='mule-demo' -Dversion=$POM_VERSION -Dpackaging=zip -Dfile=$zip_file -DgeneratePom=true"

					}

				}
				if (deployServer) {
					stage('Deploy To CloudHub') {
						env.APP_NAME = "${MULE_ENVIRONEMENT.toLowerCase()}-$MAVEN_ARTIFACTID"
						def APPLICATION_NAME = "$APP_NAME-$APP_VERSION"
						def PROPERTIES = app_properties()

						//echo "output=$PROPERTIES"

							withCredentials([usernamePassword(credentialsId: 'server', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
								sh "set +x"
								mysh "mvn clean mule:deploy -U -s $MAVEN_SETTINGS -Dmule.username=${USERNAME} -Dmule.password=${PASSWORD} -Dmule.applicationName=${MULE.APPLICATION.NAME} -Dmule.environment=${MULE.ENVIRONMENT} ${MULE.DEPLOY}"

							}
						}
					}


			
		} 
		else {
			stage('Copy Artifact') {
				def PROJECT_NAME = 'build-test-$MAVEN_ARTIFACTID'
				copyArtifacts filter: 'target/*.jar,pom.xml', fingerprintArtifacts: true, projectName: PROJECT_NAME, selector: lastSuccessful()
			}
			
			stage('Deploy To CloudHub') {
				def POM_VERSION = version()
				def INSTALL_FILE_NAME = "$MAVEN_ARTIFACTID-$POM_VERSION"
				env.APP_NAME = "${MULE_ENVIRONEMENT.toLowerCase()}-$MAVEN_ARTIFACTID"
				def APPLICATION_NAME = "$APP_NAME-$APP_VERSION"
				def PROPERTIES = app_properties()
				
				
					withCredentials([usernamePassword(credentialsId: 'server', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
						sh "set +x"
						mysh "mvn mule:deploy -P cloudHub -Dmule.artifact='target//$INSTALL_FILE_NAME-mule-application.jar' $MULE_DEPLOY -DexecutionPhase=deploy -DexecutionId=deploy -Dmule.uri=https://anypoint.mulesoft.com -Dmaven.test.skip=true -DskipMunitTests -Dmule.username=$USERNAME -Dmule.password=$PASSWORD  -Dmule.environment=$MULE_ENVIRONEMENT -Dmule.applicationName=$APPLICATION_NAME -Dbusiness.group.client.id=$CLIENT_ID -Dbusiness.group.client.secret=$CLIENT_SECRET $PROPERTIES"
						
					}
				}
			}
		
	}
	

	
}
