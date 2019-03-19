env.DISPLAY=":0"
node(label: 'master') {
    def BUILD_BRANCH = "dev" // this should eventually come from a parameter or evaluated based on the parameter
  	def MAJOR_MINOR = "${params.RELEASE_VERSION}" //eventually this may come from the version.txt file (or other) which is in the root
    def VERSION_IMAGE_LABEL = "${MAJOR_MINOR}.${BUILD_NUMBER}-${BUILD_BRANCH}" //use semantic versioning
    def LATEST_DOCKER_IMAGE_LABEL = "latest-dev"
    def IMAGE_NAME = "employee-microservice-node"
    def DOCKER_REPO = "docker-dev.artifactory.sapient.com"
    def WORKING_BRANCH = ""
	def STREAM = "${params.STREAM}"

    currentBuild.result = "SUCCESS"
    try {
		stage('Checkout'){
			branch = "${params.BRANCH_SPECIFIER}"
			WORKING_BRANCH = "${params.BRANCH_SPECIFIER}"
			echo "Branch Specifier is ${branch}"
			checkout([$class: 'GitSCM', branches: [[name: "${branch}"]], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'a6e6d971-7ca6-4e96-8c2e-f173c57c5a0c', url: 'https://github.com/amanfil/employee-microservice-node']]])
		}
		
    stage('Build'){
        def workspace = pwd ()
        sh "logoutdocker"
	      sh "logindocker"
	      sh "docker pull docker.artifactory.sapient.com/util-node:9.5.0-latest"
        sh "docker run --rm -v ${workspace}:/tmp/employee-microservice-node:Z -w /tmp/employee-microservice-node docker.artifactory.sapient.com/util-node:9.5.0-latest npm install"
    }
    stage('Run Test Cases'){
        sh "docker run --rm -v ${workspace}:/tmp/employee-frontend:Z -w /tmp/employee-frontend -e CHROME_BIN=/usr/bin/chromium-browser docker.artifactory.sapient.com/util-node:9.5.0-latest npm run test --code-coverage --watch=false"
	   }
      
      stage("Run Sonar Analysis"){
	     withSonarQubeEnv('sonarqube'){
           sh "docker run --rm -v ${workspace}:/tmp/ap-frontend -w /tmp/ap-frontend -e SONAR_HOST_URL=${SONAR_HOST_URL} -e SONAR_AUTH_TOKEN=${SONAR_AUTH_TOKEN} docker.artifactory.sapient.com/util-sonar-runner:latest /opt/sonar-scanner/bin/sonar-scanner -Dsonar.host.url=${SONAR_HOST_URL} -Dsonar.login=${SONAR_AUTH_TOKEN} -Dsonar.projectKey=sap-frontend -Dsonar.projectName=sap-frontend -Dsonar.typescript.lcov.reportPaths=coverage/lcov.info -Dsonar.typescript.jstestdriver.coveragefile=coverage/lcov.info -Dsonar.projectBaseDir=. -Dsonar.sources=src -Dsonar.password= "

		   }
	 }*/
}
    catch (err) {
        currentBuild.result = "FAILURE"
        throw err
    }

}
