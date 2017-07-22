#!groovy


node {
    
try {

emailext body: '''The dev deployment build in jenkins has been ${BUILD_CAUSE} to deploy the following artifacts from ${BRANCH_NAME} branch.

Webapp = ${webapp}
ava = ${ava}
locker.adapter = ${locker_adapter}
version.adapter = ${version_adapter}
signature.adapter = ${signature_adapter}
provision = ${provision}
computejob = ${computejob}

The above artifacts with true value will be killed and restarted in the dev. If you are in middle of something, Please navigate to the following URL and cancel the build ASAP.

${BUILD_URL}

You will be notified when the deployment is completed.''', subject: 'Jenkins Job started for Dev Deployment', to: 'pyde_venkatesh@network.lilly.com, singh_ujjwal@network.lilly.com, chennareddy_lavanya@network.lilly.com, conaway_joel_e@network.lilly.com, singireddy_naveen@network.lilly.com, fleig_dave_c@lilly.com, j.rees@lilly.com'

slackSend color: '#32CD32', message: "The dev deployment build in jenkins has been started to deploy the following artifacts from $BRANCH_NAME branch. \n\nWebapp = $webapp \nava = $ava \nlocker.adapter = ${env.locker_adapter} \nversion.adapter = $version_adapter \nsignature.adapter = $signature_adapter \nprovision = $provision \ncomputejob = $computejob  \nThe above artifacts with true value will be killed and restarted in the dev. If you are in middle of something, Please navigate to the following URL and cancel the build ASAP. \n$BUILD_URL  \nYou will be notified when the deployment is completed."
  //Workspace for the build
  ws("workspace/dev")  {

step([$class: 'WsCleanup']) 

    // Mark the code checkout 'stage'....
    stage 'Checkout'
    checkout([$class: 'GitSCM', branches: [[name: '${BRANCH_NAME}']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '118fae8c-7897-4a13-846c-036ce473322b', url: 'git@github.com:EliLillyCo/GSS_LRLIT_CLUWE-WebTool.git']]])

// Mark the code build 'stage'....
    stage 'Build'
    // Run the maven build
    sh "mvn spring-boot:build-info clean install"
   
  
    // Mark the Archieve 'stage'....
    stage 'Archieve artifacts'
    //Archieve the artifacts
    archiveArtifacts 'ui/webapp/target/*.war, service/locker.adapter/target/*.jar, service/signature.adapter/target/*.jar,  service/version.adapter/target/*.jar, service/provision/target/*.jar, service/computejob/target/*.jar, service/ava/target/*.jar'
    
 stage 'Deployment Approval'
    
     
        emailext body: '''The build has been completed for your dev deployment request in jenkins. If you want to proceed with the deployment, Please navigate to the build page and approve the deployment.

${JOB_URL}''', recipientProviders: [[$class: 'RequesterRecipientProvider']], subject: 'Approval for your dev deployment job in jenkins'

  
        input 'Do you want to proceed with the deployment?'
       
  
 

    stage('Transfer Artifacts')
        {
        sh "/home/cluwe_services_q/scripts/err/devartifacts.sh"
        }



if (webapp) {
    stage('d1-deploy')
      {
        sh "/home/cluwe_services_q/scripts/err/d1-deploy.sh"
      }
}

if (locker_adapter || version_adapter || signature_adapter || provision || ava) {

    stage('d3-deploy')  
        {
        sh "/home/cluwe_services_q/scripts/err/d3-deploy.sh"
      }
}

if (locker_adapter || version_adapter || signature_adapter || provision || ava) {
    stage('d4-deploy')
    {
        sh "/home/cluwe_services_q/scripts/err/d4-deploy.sh"
      }
      
}

if (computejob) {
    stage('d5-deploy')
    {
        sh "/home/cluwe_services_q/scripts/err/d5-deploy.sh"
      }
      
}

stage('Notification') {
			   
			currentBuild.result = "SUCCESS"
			emailext body: '''The dev deployment build in jenkins has been completed.

Webapp = ${webapp}
ava = ${ava}
locker.adapter = ${locker_adapter}
version.adapter = ${version_adapter}
signature.adapter = ${signature_adapter}
provision = ${provision}
computejob = ${computejob}

The above artifacts with true value has been succesfully deployed in dev.''', subject: 'Dev Deployment completed', to: 'pyde_venkatesh@network.lilly.com, singh_ujjwal@network.lilly.com, chennareddy_lavanya@network.lilly.com, conaway_joel_e@network.lilly.com, singireddy_naveen@network.lilly.com, fleig_dave_c@lilly.com, j.rees@lilly.com'

slackSend color: '#32CD32', message: "The dev deployment build in jenkins has been completed. \nWebapp = $webapp \nava = $ava \nlocker.adapter = ${env.locker_adapter} \nversion.adapter = $version_adapter \nsignature.adapter = $signature_adapter \nprovision = $provision \ncomputejob = $computejob  \nThe above artifacts with true value are succesfully deployed in dev."
}
  }
  }
catch (err) {
        
        String error = "${err}";
        currentBuild.result = "FAILURE"
        
        emailext body: '''The build for dev deployment in jenkins has been failed. Please navigate to the build page and check the console output for errors.

${BUILD_URL}''', recipientProviders: [[$class: 'RequesterRecipientProvider']], subject: 'FAILURE of dev deployment job in jenkins'

slackSend color: '#32CD32', message: "The build for dev deployment in jenkins has been failed. Please navigate to the build page and check the console output for errors.\n${BUILD_URL}"
  throw err
  
}
    
}
