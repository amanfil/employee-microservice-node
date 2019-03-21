env.DISPLAY=":0"
node(label: 'master') {
    def BUILD_BRANCH = "dev" // this should eventually come from a parameter or evaluated based on the parameter
  	def MAJOR_MINOR = "${params.RELEASE_VERSION}" //eventually this may come from the version.txt file (or other) which is in the root
    def VERSION_IMAGE_LABEL = "${MAJOR_MINOR}.${BUILD_NUMBER}-${BUILD_BRANCH}" //use semantic versioning
    def LATEST_DOCKER_IMAGE_LABEL = "latest-dev"
    def IMAGE_NAME = "employee-microservice-node"
    def DOCKER_REPO = "docker.artifactory.sapient.com"
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
        sh "docker pull docker.artifactory.sapient.com/util-sonar-runner:latest"
	     withSonarQubeEnv('sonarqube'){
           sh "docker run --rm -v ${workspace}:/tmp/employee-frontend -w /tmp/employee-frontend -e SONAR_HOST_URL=${SONAR_HOST_URL} -e SONAR_AUTH_TOKEN=${SONAR_AUTH_TOKEN} docker.artifactory.sapient.com/util-sonar-runner:latest /opt/sonar-scanner/bin/sonar-scanner -Dsonar.host.url=${SONAR_HOST_URL} -Dsonar.login=${SONAR_AUTH_TOKEN} -Dsonar.projectKey=sap-frontend -Dsonar.projectName=sap-frontend  -Dsonar.javascript.lcov.reportPaths=/tmp/employee-frontend/coverage/index.html -Dsonar.javascript.jstestdriver.coveragefile=/tmp/employee-frontend/coverage/index.html -Dsonar.projectBaseDir=. -Dsonar.sources=app -Dsonar.password= "

		   }
	 }

   stage("Quality Gate"){
	  sh "sleep 30s"
      withSonarQubeEnv('sonarqube') {
        env.SONAR_CE_TASK_URL = sh(returnStdout: true, script: """cat ${workspace}/.sonar/report-task.txt|grep -a 'ceTaskUrl'|awk -F '=' '{print \$2\"=\"\$3}'""")
        timeout(time: 1, unit: 'MINUTES') {
            sh 'curl -u $SONAR_AUTH_TOKEN $SONAR_CE_TASK_URL -o ceTask.json'
            env.analysisID = sh(returnStdout: true, script: """cat ceTask.json |awk -F 'analysisId' '{print \$2}'|awk -F ':' '{print \$2}'|awk -F '\"' '{print \$2}'""")
            sh "echo $analysisID"
            println(analysisID)
            env.qualityGateUrl = env.SONAR_HOST_URL + "/api/qualitygates/project_status?analysisId=" + env.analysisID
            sh 'curl -u $SONAR_AUTH_TOKEN $qualityGateUrl -o qualityGate.json'
            env.qualitygate = sh(returnStdout: true, script: """cat qualityGate.json |awk -F 'status' '{print \$2}'|awk -F ':' '{print \$2}'|awk -F '\"' '{print \$2}'""")
            if (qualitygate.trim().equals("ERROR")) {
              error  "Quality Gate failure"
            }
            echo  "Quality Gate success"
          }
        }
      }
      stage('Build Docker Image'){
      sh "logoutdocker"
      sh "logindocker"
      sh "cd ${workspace}"
	    if(WORKING_BRANCH =~ 'origin/master') {
            VERSION_IMAGE_LABEL = "${MAJOR_MINOR}.${BUILD_NUMBER}-dev"
            LATEST_DOCKER_IMAGE_LABEL = "latest-dev${STREAM}"
        }
        
		sh "echo 'the current version is [${VERSION_IMAGE_LABEL}] the latest tag is [${LATEST_DOCKER_IMAGE_LABEL}] from branch [${WORKING_BRANCH}] - ${params.BRANCH_SPECIFIER}'"
        sh "docker build -t ${DOCKER_REPO}/${IMAGE_NAME}:${VERSION_IMAGE_LABEL} -f Dockerfile ."
      }
      stage ('Tag Docker Image') {
      	    sh "docker tag ${DOCKER_REPO}/${IMAGE_NAME}:${VERSION_IMAGE_LABEL} ${DOCKER_REPO}/${IMAGE_NAME}:${LATEST_DOCKER_IMAGE_LABEL}"
	   }
   }   
    catch (err) {
        currentBuild.result = "FAILURE"
        throw err
    }

}
