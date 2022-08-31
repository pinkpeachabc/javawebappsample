import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=6015df2b-244a-424f-8d75-5aad1eee0020',
        'AZURE_TENANT_ID=c023101b-be0d-4a03-991a-824f9032469a']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
      sh 'docker build -t myapp .'
    }
  
    stage('deploy') {
      def resourceGroup = 'jenkins-get-started-rg'
      def webAppName = 'Jenkins-test'
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'f82172eb-82bc-442d-8243-bf0538dbb0e0', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
       sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }
      // get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson
      // upload package
      // sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      // 把docker 发布到 app service
      sh "az webapp config container set -g $resourceGroup -n $webAppName --docker-custom-image-name myapp --docker-registry-server-url https://index.docker.io/v1/ --docker-registry-server-user $DOCKER_USERNAME --docker-registry-server-password $DOCKER_PASSWORD"
      // log out
      sh 'az logout'
    }
  }
}
