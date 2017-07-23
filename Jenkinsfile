#!groovy

/*
#################### 
# Jenkins Pipeline script to build and deploy artifacts in DEV environment
# This script accepts Git branch name, Username, Password, cluwe services (yes or no) as input parametres
# This script checkout code from selected git branch, build the code, SSH into dev servers, transfer the artifacts 
# into the servers, stop the running artifacts, takes backup of stopped artifacts and logs and start the transferred
# artifacts and do the verification
# This script notifies the team upon startup and completion of the job.
# This job requires approval from any of the team member after the build to proceed with the deployment.
#
# Copyright (c) 2016 Eli Lilly and Company
# 
# All rights reserved
#
####################
*/

node {
	try {
		
		// Send the e-mail to the team to notify the start of deployment build
		emailext body: '''The dev deployment build in jenkins has been ${BUILD_CAUSE} to deploy the following artifacts from ${BRANCH_NAME} branch./n/nWebapp = ${webapp}/nava = ${ava}/nlocker.adapter = ${locker_adapter}/nversion.adapter = ${version_adapter}/nsignature.adapter = ${signature_adapter}/nprovision = ${provision}/ncomputejob = ${computejob}/n/nThe above artifacts with true value will be killed and restarted in the dev. If you are in middle of something, Please navigate to the following URL and cancel the build ASAP./n/n${BUILD_URL}/n/nYou will be notified when the deployment is completed.''', 
			
			subject: 'Jenkins Job started for Dev Deployment', 
			to: 'pyde_venkatesh@network.lilly.com'
		
		// Send the message to the Slack team channel to notify the start of deployment build
		slackSend color: '#32CD32', message: "The dev deployment build in jenkins has been started to deploy the following artifacts from $BRANCH_NAME branch. \n\nWebapp = $webapp \nava = $ava \nlocker.adapter = ${env.locker_adapter} \nversion.adapter = $version_adapter \nsignature.adapter = $signature_adapter \nprovision = $provision \ncomputejob = $computejob  \nThe above artifacts with true value will be killed and restarted in the dev. If you are in middle of something, Please navigate to the following URL and cancel the build ASAP. \n$BUILD_URL  \nYou will be notified when the deployment is completed."
	
		
		// workspace for the code
		ws("workspace/dev")
		{
			// Clean up the workspace before checking out code
			step([$class: 'WsCleanup']) 
			
			// Mark the code checkout 'stage'.... This stage check out code from selected GIT branch into the workspace
			stage ('Checkout') {
			checkout([$class: 'GitSCM', branches: [[name: '${BRANCH_NAME}']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '118fae8c-7897-4a13-846c-036ce473322b', url: 'git@github.com:EliLillyCo/GSS_LRLIT_CLUWE-WebTool.git']]])
			}
			
			// Extract Maven pom version from pom.xml file
			pom = readMavenPom file: 'pom.xml'
			pom.version
			env.pom_version = pom.version
			
			// Mark the code build 'stage'.... This stage build the code
			stage 'Build'
			// Run the maven build
			sh "mvn spring-boot:build-info clean install -DskipTests"
			
			
			// Mark the Archieve 'stage'....
			stage 'Archieve artifacts'
			//Archieve the artifacts
			archiveArtifacts 'ui/webapp/target/*.war, service/locker.adapter/target/*.jar, service/signature.adapter/target/*.jar,  service/version.adapter/target/*.jar, service/provision/target/*.jar, service/computejob/target/*.jar, service/ava/target/*.jar'
			
			// This stage pause the job after build to proceed with the deployment. It waits for approval from any of the team member.
			stage 'Deployment Approval'
    
			// E-mail will be sent to the user who started this job asking for approval for the deployment.
			emailext body: '''The build has been completed for your dev deployment request in jenkins. If you want to proceed with the deployment, Please navigate to the build page and approve the deployment.\n\n${JOB_URL}''', 
				recipientProviders: [[$class: 'RequesterRecipientProvider']], 
				subject: 'Approval for your dev deployment job in jenkins'
			
			// This input step provides 'yes' or 'no' option for Deployment Approval.
			input 'Do you want to proceed with the deployment?'
			
			
			// This stage runs the devartifacts script that transfers the artifacts selected for deployment by doing SSH into servers using provided username and password
			stage('Transfer Artifacts')
			{
				sh "/home/cluwe_services_q/scripts/final/devartifacts.sh"
			}
			
			
			// This stage runs only when webapp parameter is selected.
			if (params.webapp) 
			{
				// This stage runs the d1-deploy script that deploys webapp in d1 server.
				stage('d1-deploy')
				{
				    sh "echo ${pom.version}"
					sh "/home/cluwe_services_q/scripts/final/d1-deploy.sh"
				}
			}
			
			
			// This stage runs only when atleast one of the d3 services is selected.
			if (params.locker_adapter || params.version_adapter || params.signature_adapter || params.provision || params.ava)
			{
				// This stage runs the d3-deploy script that deploy selected services in d3 server.
				stage('d3-deploy')
				{
					sh "/home/cluwe_services_q/scripts/final/d3-deploy.sh"
				}
			}
			
			
			// This stage runs only when atleast one of the d4 services is selected.
			if (params.locker_adapter || params.version_adapter || params.signature_adapter || params.provision || params.ava)
			{
				// This stage runs the d4-deploy script that deploy selected services in d4 server.
				stage('d4-deploy')
				{
					sh "/home/cluwe_services_q/scripts/final/d4-deploy.sh"
				}
			}
			
			
			// This stage runs only when computejob parameter is selected.
			if (params.computejob)
			{
				// This stage runs the d5-deploy script that deploys computejob service in d5 server.
				stage('d5-deploy')
				{
					sh "/home/cluwe_services_q/scripts/final/d5-deploy.sh"
				}
			}
			
			
			// This stage notifies team upon successfull deployment.
			stage('Notification') {
			   
			currentBuild.result = "SUCCESS"
				
				// Send the e-mail to the team to notify the completion of deployment.
				emailext body: '''The dev deployment build in jenkins has been completed.\nWebapp = ${webapp}\nava = ${ava}\nlocker.adapter = ${locker_adapter}\nversion.adapter = ${version_adapter}\nsignature.adapter = ${signature_adapter}\nprovision = ${provision}\ncomputejob = ${computejob}\n\nThe above artifacts with true value has been succesfully deployed in dev.''', 
					
					subject: 'Dev Deployment completed',
					to: 'pyde_venkatesh@network.lilly.com'
				
				// Send the message to the Slack team channel to notify the completion of deployment.
				slackSend color: '#32CD32', message: "The dev deployment build in jenkins has been completed. \nWebapp = $webapp \nava = $ava \nlocker.adapter = ${env.locker_adapter} \nversion.adapter = $version_adapter \nsignature.adapter = $signature_adapter \nprovision = $provision \ncomputejob = $computejob  \nThe above artifacts with true value are succesfully deployed in dev."
			
			}
		}
	}
	
	catch (err) {
		String error = "${err}";
		currentBuild.result = "FAILURE"
		
		// Send the e-mail to the user to notify the error in deployment.
		emailext body: '''The dev deployment job in jenkins has been failed. Please navigate to the build page and check the console output for errors./n/n${BUILD_URL}''', 
			
			recipientProviders: [[$class: 'RequesterRecipientProvider']], 
			subject: 'FAILURE of dev deployment job in jenkins'
		
		
		slackSend color: '#32CD32', message: "The build for dev deployment in jenkins has been failed. Please navigate to the build page and check the console output for errors.\n${BUILD_URL}"
		
		throw err
	}
}
