import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=a8cddc3a-fd61-452d-8abe-f7b44b38f5fd',
        'AZURE_TENANT_ID=aef88f84-f357-44a2-a334-5245d3f2adf9']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'cloud-shell-storage-centralindia'
      def webAppName = 'java460'
      // login Azure
      withCredentials([usernamePassword(credentialsId: '1931ff2b-5ad0-4b2e-9a2c-b45fc6d1469e', passwordVariable: 'HOj8Q~h2.9JxFspqj4jOP6b_yZCRSg.lnPUDxaPw', usernameVariable: 'e8f3dd81-fbba-48da-a825-f2d7bdb407c1')]) {
       sh '''
          az login --service-principal -u 0ffda873-3dc5-4047-abdd-7a246f7437da -p HOj8Q~h2.9JxFspqj4jOP6b_yZCRSg.lnPUDxaPw -t aef88f84-f357-44a2-a334-5245d3f2adf9
          az account set -s a8cddc3a-fd61-452d-8abe-f7b44b38f5fd
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
