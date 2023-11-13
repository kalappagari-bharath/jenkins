import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['1bb1f18e-8f6f-4757-aa78-5fd3dd405b99',
        'AZURE_TENANT_ID=89627a4e-a49f-415a-b6fd-acd74f822cb6']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'Reddy460_group'
      def webAppName = 'Reddy460'
      // login Azure
      withCredentials([usernamePassword(credentialsId: '215b2f35-3eaf-417d-9d84-565ea6fa992d', passwordVariable: 'Ggn8Q~tKt.TbNsi_gql5KXAy1L6JErx5UvEH7ats', usernameVariable: '1d28e5a9-13de-4809-a7be-2a35385998c1')]) {
       sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
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
