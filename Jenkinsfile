import groovy.json.JsonSlurper

def getAcrLoginServer(def acrSettingsJson) {
  def acrSettings = new JsonSlurper().parseText(acrSettingsJson)
  return acrSettings.loginServer
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
      def webAppResourceGroup = 'app-docker'
      def webAppName = 'JavaWebApp-kylie-docker'
      def acrName = 'kylieacr'
      def imageName = 'calculator'
      // generate version, it's important to remove the trailing new line in git describe output
      def version = sh script: 'git describe | tr -d "\n"', returnStdout: true
      withCredentials([usernamePassword(credentialsId: '51c01e79-26fb-4480-877f-82c968114792', passwordVariable: 'rMW1u.-gWEloBfTCJmyk0HOoMWLLodI-xJ', usernameVariable: '59b697c7-fa45-41df-9829-ba48ed116cdc')]) {
        // login Azure
        sh '''
          az login --service-principal -u 59b697c7-fa45-41df-9829-ba48ed116cdc -p rMW1u.-gWEloBfTCJmyk0HOoMWLLodI-xJ -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
         // get login server
        def acrSettingsJson = sh script: "az acr show -n $acrName", returnStdout: true
        def loginServer = getAcrLoginServer acrSettingsJson
        // login docker
        // docker.withRegistry only supports credential ID, so use native docker command to login
        // you can also use docker.withRegistry if you add a credential
        sh "docker login -u 59b697c7-fa45-41df-9829-ba48ed116cdc -p rMW1u.-gWEloBfTCJmyk0HOoMWLLodI-xJ $loginServer"
        // build image
        def imageWithTag = "$loginServer/$imageName:$version"
        def image = docker.build imageWithTag
        // push image
        image.push()
        // update web app docker settings
        sh "az webapp config container set -g $webAppResourceGroup -n $webAppName -c $imageWithTag -r http://$loginServer -u 59b697c7-fa45-41df-9829-ba48ed116cdc -p rMW1u.-gWEloBfTCJmyk0HOoMWLLodI-xJ"
        // log out
        sh 'az logout'
        sh "docker logout $loginServer"
      }
    }
  }
}
