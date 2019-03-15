env.DISPLAY=":0"
node(label: 'slave') {
    def BUILD_BRANCH = "dev" // this should eventually come from a parameter or evaluated based on the parameter
  	def MAJOR_MINOR = "${params.RELEASE_VERSION}" //eventually this may come from the version.txt file (or other) which is in the root
    def VERSION_IMAGE_LABEL = "${MAJOR_MINOR}.${BUILD_NUMBER}-${BUILD_BRANCH}" //use semantic versioning
    def LATEST_DOCKER_IMAGE_LABEL = "latest-dev"
    def IMAGE_NAME = "cp-experience-document-service"
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
		
/*		stage('Gradle Build'){
			def workspace = pwd()
			sh "logoutdocker"
			sh "logindocker"
			sh "rm -rf ${workspace}/build"
			sh "docker pull docker.artifactory.sapient.com/util-gradle:latest"
			sh "docker run --rm -v ${workspace}:/home/gradle -v gradle_cache:/home/gradle/.gradle -u `id -u`:`id -g` -w /home/gradle docker.artifactory.sapient.com/util-gradle:latest gradle build"
		}      

		stage ('Run Test Cases') {
			junit 'build/test-results/**/*.xml'
		}
	
		stage("Run Fortify Security Analysis"){
			if(FORTIFY_RUN == 'true'){
				sh "docker pull docker.artifactory.sapient.com/util-fortify-sca:1.0.2"
				sh "docker run --rm -v ${workspace}:/home/gradle -v gradle_cache:/home/gradle/.gradle -u `id -u`:`id -g` -w /home/gradle  docker.artifactory.sapient.com/util-fortify-sca:1.0.2 sh -c 'export PATH=$PATH:/opt/fortify/bin:/opt/gradle-4.6/bin;fortifyupdate;sourceanalyzer -64 -b ${IMAGE_NAME} -cp /root/.gradle/**/*.jar -source 1.8 gradle clean build;sourceanalyzer -64 -b ${IMAGE_NAME} -cp /root/.gradle/**/*.jar -source 1.8 -gradle -verbose gradle assemble;sourceanalyzer -b ${IMAGE_NAME} -scan -64 -cp /root/.gradle/**/*.jar -f fortifyResults-${IMAGE_NAME}.fpr'"
				
				sh "docker run --rm -v ${workspace}:/home/gradle -v gradle_cache:/home/gradle/.gradle -u `id -u`:`id -g` -w /home/gradle  docker.artifactory.sapient.com/util-fortify-sca:1.0.2 /opt/fortify/bin/FPRUtility -project fortifyResults-${IMAGE_NAME}.fpr -information -search -query \"[fortify priority order]:critical\" -categoryIssueCounts -listIssues > fortifyResults-${IMAGE_NAME}-critical.txt"
				
				sh "docker run --rm -v ${workspace}:/home/gradle -v gradle_cache:/home/gradle/.gradle -u `id -u`:`id -g` -w /home/gradle  docker.artifactory.sapient.com/util-fortify-sca:1.0.2 /opt/fortify/bin/FPRUtility -project fortifyResults-${IMAGE_NAME}.fpr -information -search -query \"[fortify priority order]:high\" -categoryIssueCounts -listIssues > fortifyResults-${IMAGE_NAME}-high.txt"
				
				sh "docker run --rm -v ${workspace}:/home/gradle -v gradle_cache:/home/gradle/.gradle -u `id -u`:`id -g` -w /home/gradle  docker.artifactory.sapient.com/util-fortify-sca:1.0.2 /opt/fortify/bin/ReportGenerator -format pdf -f fortifyResults-${IMAGE_NAME}.pdf -source fortifyResults-${IMAGE_NAME}.fpr"

                archiveArtifacts "fortifyResults-${IMAGE_NAME}.pdf"
				archiveArtifacts "fortifyResults-${IMAGE_NAME}-critical.txt"
				archiveArtifacts "fortifyResults-${IMAGE_NAME}-high.txt"
            }
        }
		
		stage("Run Sonar Analysis"){
			sh "docker pull docker.artifactory.sapient.com/util-sonar-runner:latest"
			withSonarQubeEnv('sonarqube'){
				sh "docker run --rm -v ${workspace}:/opt/employee-microservice-node -w /opt/employee-microservice-node -e SONAR_HOST_URL=${SONAR_HOST_URL} -e SONAR_AUTH_TOKEN=${SONAR_AUTH_TOKEN} docker.artifactory.sapient.com/util-sonar-runner:latest /opt/sonar-scanner/bin/sonar-scanner -Dsonar.host.url=${SONAR_HOST_URL} -Dsonar.login=${SONAR_AUTH_TOKEN} -Dsonar.projectKey=cetera-employee-microservice-node -Dsonar.projectName=cetera-employee-microservice-node  -Dsonar.projectBaseDir=. -Dsonar.sources=./src -Dsonar.java.binaries=./build/classes -Dsonar.junit.reportPaths=./build/test-results/test -Dsonar.jacoco.reportPaths=./build/jacoco/test.exec -Dsonar.fortify.reportPath=fortifyResults-${IMAGE_NAME}.fpr -Dsonar.password="
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
	    if(WORKING_BRANCH =~ 'origin/development') {
            VERSION_IMAGE_LABEL = "${MAJOR_MINOR}.${BUILD_NUMBER}-dev"
            LATEST_DOCKER_IMAGE_LABEL = "latest-dev${STREAM}"
        }
        if(WORKING_BRANCH =~ 'origin/stable' || WORKING_BRANCH =~ 'origin/release/v') {
            VERSION_IMAGE_LABEL = "${MAJOR_MINOR}.${BUILD_NUMBER}-stable"
            LATEST_DOCKER_IMAGE_LABEL = "latest-stable"
        }
		sh "echo 'the current version is [${VERSION_IMAGE_LABEL}] the latest tag is [${LATEST_DOCKER_IMAGE_LABEL}] from branch [${WORKING_BRANCH}] - ${params.BRANCH_SPECIFIER}'"
        sh "echo ${VERSION_IMAGE_LABEL} > version.txt"
        sh "docker build -t ${DOCKER_REPO}/${IMAGE_NAME}:${VERSION_IMAGE_LABEL} -f Dockerfile ."
      }
      stage ('Tag Docker Image') {
      	    sh "docker tag ${DOCKER_REPO}/${IMAGE_NAME}:${VERSION_IMAGE_LABEL} ${DOCKER_REPO}/${IMAGE_NAME}:${LATEST_DOCKER_IMAGE_LABEL}"
	  }
   stage('Docker Image Upload to Artifactory'){
      sh "logoutdocker"
      sh "login.sh dev pub"
        if(WORKING_BRANCH =~ 'origin/development' || WORKING_BRANCH =~ 'origin/stable' || WORKING_BRANCH =~ 'origin/release/v') {
            sh "docker push ${DOCKER_REPO}/${IMAGE_NAME}:${VERSION_IMAGE_LABEL}"
            sh "docker push ${DOCKER_REPO}/${IMAGE_NAME}:${LATEST_DOCKER_IMAGE_LABEL}"
        }
      sh "logoutdocker"
      sh "docker rmi -f ${DOCKER_REPO}/${IMAGE_NAME}:${VERSION_IMAGE_LABEL}"
      sh "docker rmi -f ${DOCKER_REPO}/${IMAGE_NAME}:${LATEST_DOCKER_IMAGE_LABEL}"
	}
	stage ('Git Tag') {
		if(WORKING_BRANCH =~ 'origin/stable' || WORKING_BRANCH =~ 'origin/release/v') {
			sh "git tag -d ${VERSION_IMAGE_LABEL} || true"
			sh "git tag -a ${VERSION_IMAGE_LABEL} -m \"New Tag from Stable\""
			sh "git push --tags"
		}
	}*/
	
}
    catch (err) {
        currentBuild.result = "FAILURE"
        throw err
    }
/* finally{
        if(currentBuild.result=='SUCCESS'){
			if(WORKING_BRANCH == 'origin/development1.5') {
				stage('Deploy to DEV') {
					build job: 'ACP-Deploy-From-Cetera-Repos/document-services-Cetera-deployment', wait: false, parameters: [[$class: 'StringParameterValue', name: 'ENVIRONMENT', value: "dev"]]
				}
			}
		if(WORKING_BRANCH == 'origin/stable' || WORKING_BRANCH == 'origin/release/v1') {
			}

        }
    }   */

}
