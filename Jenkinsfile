@Library('sapient-shared-library@master') _
env.DISPLAY=":0"
node(label: 'master') {
    env.BUILD_BRANCH = "dev" // this should eventually come from a parameter or evaluated based on the parameter
  	env.MAJOR_MINOR = 1.0 //eventually this may come from the version.txt file (or other) which is in the root
    env.VERSION_IMAGE_LABEL = "${MAJOR_MINOR}.${BUILD_NUMBER}-${BUILD_BRANCH}" //use semantic versioning
    env.LATEST_DOCKER_IMAGE_LABEL = "latest-dev"
    env.IMAGE_NAME = "employee-microservice-node"
    env.DOCKER_REPO = "docker.artifactory.sapient.com"
    env.WORKING_BRANCH = ""
	

    currentBuild.result = "SUCCESS"
    try {
		stage('Checkout'){
			  Git{}
		}
		
    stage('Build'){
        npm_Build{}
    }
    stage('Run Test Cases'){
        npm_Test{}
	   }
      
      stage("Run Sonar Analysis"){
        gradle_SonarAnalysis{}

		   }
	 }

     stage("Quality Gate"){
          Sonar_QualityGate{}
	    }
  
      stage('Build Docker Image'){
          build_Image()
        }
        
		
      stage ('Tag Docker Image') {
      	   tag_Image{}
	   }
      
     stage('Docker Image Upload to Artifactory'){
           upload_Image{}
   }   
} 
    catch (err) {
        currentBuild.result = "FAILURE"
        throw err
    }

}
