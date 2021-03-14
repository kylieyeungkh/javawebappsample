import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=4db6e230-5559-4603-af25-b076991b28eb',
        'AZURE_TENANT_ID=443a6b86-9380-4794-9072-90960acb40e5']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'app'
      def webAppName = 'JavaWebApp-kylie'
      // login Azure
      withCredentials([usernamePassword(credentialsId: '51c01e79-26fb-4480-877f-82c968114792', passwordVariable: 'rMW1u.-gWEloBfTCJmyk0HOoMWLLodI-xJ', usernameVariable: '59b697c7-fa45-41df-9829-ba48ed116cdc')]) {
       sh '''
          az login --service-principal -u 59b697c7-fa45-41df-9829-ba48ed116cdc -p rMW1u.-gWEloBfTCJmyk0HOoMWLLodI-xJ -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }
      // get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson
      // upload package
      sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      // log out
      sh 'az logout'
    }
  }
}
